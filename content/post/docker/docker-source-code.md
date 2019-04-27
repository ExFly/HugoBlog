---
title: "docker源码分析"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: []
date: 2019-04-23T21:35:46+08:00
---

文章简介：`docker`代码结构，`docker run`代码分析

<!--more-->

# docker 全局架构

![docker架构图](/media/img/docker/docker-architecture.jpg)

## 模块

DockerClient DockerDaemon DockerRegistry Graph Driver libcontainer DockerContainer

libcontainer 涉及大量 linux 内核特性，包括 namespaces,cgroups,capabilities

### 各部分关联

通信方式 tcp://host:port unix://path_to_socket fd://socketfd

[from here](https://www.huweihuang.com/kubernetes-notes/docker/docker-architecture.html)

- 用户是使用 Docker Client 与 Docker Daemon 建立通信，并发送请求给后者。
- Docker Daemon 作为 Docker 架构中的主体部分，首先提供 Server 的功能使其可以接受 Docker Client 的请求；
- Engine 执行 Docker 内部的一系列工作，每一项工作都是以一个 Job 的形式的存在。
- Job 的运行过程中，当需要容器镜像时，则从 Docker Registry 中下载镜像，并通过镜像管理驱动 graphdriver 将下载镜像以 Graph 的形式存储；
- 当需要为 Docker 创建网络环境时，通过网络管理驱动 networkdriver 创建并配置 Docker 容器网络环境；
- 当需要限制 Docker 容器运行资源或执行用户指令等操作时，则通过 execdriver 来完成。
- libcontainer 是一项独立的容器管理包，networkdriver 以及 execdriver 都是通过 libcontainer 来实现具体对容器进行的操作。

具体的可以看[这里](https://www.huweihuang.com/article/docker/docker-architecture/)，已经介绍的很清楚。

## `docker run`执行流程分析

整体执行流程: 从本地镜像中寻找是否存在命令指定镜像，如果存在则正常返回，执行接下来流程。如果不存在镜像，deamon 返回错误，由 cli 重新发起 pull 请求，之后重试。

### 总体描述

- createContainer
- ContainerStart

代码 `lanything/code-read` forked `moby/moby`, `lanything/cli` forked `docker/cli`

### createContainer

- `docker/cli` cli/command/container/run.go#NewRunCommand
- `docker/cli` cli/command/container/run.go#runContainer

```go
// cli/command/container/create.go#createContainer
func createContainer(ctx context.Context, dockerCli command.Cli, containerConfig *containerConfig, opts *createOptions) (*container.ContainerCreateCreatedBody, error) {
	config := containerConfig.Config
	hostConfig := containerConfig.HostConfig
	networkingConfig := containerConfig.NetworkingConfig
	stderr := dockerCli.Err()

	warnOnOomKillDisable(*hostConfig, stderr)
	warnOnLocalhostDNS(*hostConfig, stderr)

	var (
		trustedRef reference.Canonical
		namedRef   reference.Named
	)

	containerIDFile, err := newCIDFile(hostConfig.ContainerIDFile)
	if err != nil {
		return nil, err
	}
	defer containerIDFile.Close()

	ref, err := reference.ParseAnyReference(config.Image)
	if err != nil {
		return nil, err
	}
	if named, ok := ref.(reference.Named); ok {
		namedRef = reference.TagNameOnly(named)

		if taggedRef, ok := namedRef.(reference.NamedTagged); ok && !opts.untrusted {
			var err error
			trustedRef, err = image.TrustedReference(ctx, dockerCli, taggedRef, nil)
			if err != nil {
				return nil, err
			}
			config.Image = reference.FamiliarString(trustedRef)
		}
	}

	//create the container
	response, err := dockerCli.Client().ContainerCreate(ctx, config, hostConfig, networkingConfig, opts.name)

	//if image not found try to pull it
	if err != nil {
		if apiclient.IsErrNotFound(err) && namedRef != nil {
			fmt.Fprintf(stderr, "Unable to find image '%s' locally\n", reference.FamiliarString(namedRef))

			// we don't want to write to stdout anything apart from container.ID
			if err := pullImage(ctx, dockerCli, config.Image, opts.platform, stderr); err != nil {
				return nil, err
			}
			if taggedRef, ok := namedRef.(reference.NamedTagged); ok && trustedRef != nil {
				if err := image.TagTrusted(ctx, dockerCli, trustedRef, taggedRef); err != nil {
					return nil, err
				}
			}
			// Retry
			var retryErr error
			response, retryErr = dockerCli.Client().ContainerCreate(ctx, config, hostConfig, networkingConfig, opts.name)
			if retryErr != nil {
				return nil, retryErr
			}
		} else {
			return nil, err
		}
	}

	for _, warning := range response.Warnings {
		fmt.Fprintf(stderr, "WARNING: %s\n", warning)
	}
	err = containerIDFile.Write(response.ID)
	return &response, err
}
```

- `docker/cli` cli/command/container/create.go#createContainer 调用 `ContainerCreate`
- `moby/moby` api/server/router/container/container.go#initRoutes.postContainersCreate
- `moby/moby` api/server/router/container/container_routes.go#postContainersCreate
- `moby/moby` daemon/create.go#ContainerCreate

### containerCreate

```go
// `moby/moby` daemon/create.go#containerCreate
func (daemon *Daemon) containerCreate(opts createOpts) (containertypes.ContainerCreateCreatedBody, error) {
	start := time.Now()
	if opts.params.Config == nil {
		return containertypes.ContainerCreateCreatedBody{}, errdefs.InvalidParameter(errors.New("Config cannot be empty in order to create a container"))
	}

	os := runtime.GOOS
	if opts.params.Config.Image != "" {
		img, err := daemon.imageService.GetImage(opts.params.Config.Image)
		if err == nil {
			os = img.OS
		}
	} else {
		// This mean scratch. On Windows, we can safely assume that this is a linux
		// container. On other platforms, it's the host OS (which it already is)
		if runtime.GOOS == "windows" && system.LCOWSupported() {
			os = "linux"
		}
	}

	warnings, err := daemon.verifyContainerSettings(os, opts.params.HostConfig, opts.params.Config, false)
	if err != nil {
		return containertypes.ContainerCreateCreatedBody{Warnings: warnings}, errdefs.InvalidParameter(err)
	}

	err = verifyNetworkingConfig(opts.params.NetworkingConfig)
	if err != nil {
		return containertypes.ContainerCreateCreatedBody{Warnings: warnings}, errdefs.InvalidParameter(err)
	}

	if opts.params.HostConfig == nil {
		opts.params.HostConfig = &containertypes.HostConfig{}
	}
	err = daemon.adaptContainerSettings(opts.params.HostConfig, opts.params.AdjustCPUShares)
	if err != nil {
		return containertypes.ContainerCreateCreatedBody{Warnings: warnings}, errdefs.InvalidParameter(err)
	}

	container, err := daemon.create(opts)
	if err != nil {
		return containertypes.ContainerCreateCreatedBody{Warnings: warnings}, err
	}
	containerActions.WithValues("create").UpdateSince(start)

	if warnings == nil {
		warnings = make([]string, 0) // Create an empty slice to avoid https://github.com/moby/moby/issues/38222
	}

	return containertypes.ContainerCreateCreatedBody{ID: container.ID, Warnings: warnings}, nil
}
```

### GetImages

```go
// GetImage returns an image corresponding to the image referred to by refOrID.
func (i *ImageService) GetImage(refOrID string) (*image.Image, error) {
	ref, err := reference.ParseAnyReference(refOrID)
	if err != nil {
		return nil, errdefs.InvalidParameter(err)
	}
	namedRef, ok := ref.(reference.Named)
	if !ok {
		digested, ok := ref.(reference.Digested)
		if !ok {
			return nil, ErrImageDoesNotExist{ref}
		}
		id := image.IDFromDigest(digested.Digest())
		if img, err := i.imageStore.Get(id); err == nil {
			return img, nil
		}
		return nil, ErrImageDoesNotExist{ref}
	}

	if digest, err := i.referenceStore.Get(namedRef); err == nil {
		// Search the image stores to get the operating system, defaulting to host OS.
		id := image.IDFromDigest(digest)
		if img, err := i.imageStore.Get(id); err == nil {
			return img, nil
		}
	}

	// Search based on ID
	if id, err := i.imageStore.Search(refOrID); err == nil {
		img, err := i.imageStore.Get(id)
		if err != nil {
			return nil, ErrImageDoesNotExist{ref}
		}
		return img, nil
	}

	return nil, ErrImageDoesNotExist{ref}
}
```

`ImageService.imageStore` 在`moby`daemon/daemon.go#NewDaemon 中有设置
`ImageService.imageStore` 在`moby`image/store.go#NewImageStore 有初始化

### ContainerStart

`moby` daemon/start.go#ContainerStart

```go
func (daemon *Daemon) ContainerStart(name string, hostConfig *containertypes.HostConfig, checkpoint string, checkpointDir string) error {
	if checkpoint != "" && !daemon.HasExperimental() {
		return errdefs.InvalidParameter(errors.New("checkpoint is only supported in experimental mode"))
	}

	container, err := daemon.GetContainer(name)
	if err != nil {
		return err
	}

	validateState := func() error {
		container.Lock()
		defer container.Unlock()

		if container.Paused {
			return errdefs.Conflict(errors.New("cannot start a paused container, try unpause instead"))
		}

		if container.Running {
			return containerNotModifiedError{running: true}
		}

		if container.RemovalInProgress || container.Dead {
			return errdefs.Conflict(errors.New("container is marked for removal and cannot be started"))
		}
		return nil
	}

	if err := validateState(); err != nil {
		return err
	}

	// Windows does not have the backwards compatibility issue here.
	if runtime.GOOS != "windows" {
		// This is kept for backward compatibility - hostconfig should be passed when
		// creating a container, not during start.
		if hostConfig != nil {
			logrus.Warn("DEPRECATED: Setting host configuration options when the container starts is deprecated and has been removed in Docker 1.12")
			oldNetworkMode := container.HostConfig.NetworkMode
			if err := daemon.setSecurityOptions(container, hostConfig); err != nil {
				return errdefs.InvalidParameter(err)
			}
			if err := daemon.mergeAndVerifyLogConfig(&hostConfig.LogConfig); err != nil {
				return errdefs.InvalidParameter(err)
			}
			if err := daemon.setHostConfig(container, hostConfig); err != nil {
				return errdefs.InvalidParameter(err)
			}
			newNetworkMode := container.HostConfig.NetworkMode
			if string(oldNetworkMode) != string(newNetworkMode) {
				// if user has change the network mode on starting, clean up the
				// old networks. It is a deprecated feature and has been removed in Docker 1.12
				container.NetworkSettings.Networks = nil
				if err := container.CheckpointTo(daemon.containersReplica); err != nil {
					return errdefs.System(err)
				}
			}
			container.InitDNSHostConfig()
		}
	} else {
		if hostConfig != nil {
			return errdefs.InvalidParameter(errors.New("Supplying a hostconfig on start is not supported. It should be supplied on create"))
		}
	}

	// check if hostConfig is in line with the current system settings.
	// It may happen cgroups are umounted or the like.
	if _, err = daemon.verifyContainerSettings(container.OS, container.HostConfig, nil, false); err != nil {
		return errdefs.InvalidParameter(err)
	}
	// Adapt for old containers in case we have updates in this function and
	// old containers never have chance to call the new function in create stage.
	if hostConfig != nil {
		if err := daemon.adaptContainerSettings(container.HostConfig, false); err != nil {
			return errdefs.InvalidParameter(err)
		}
	}
	return daemon.containerStart(container, checkpoint, checkpointDir, true)
}
```

`moby` daemon/start.go#containerStart

```go
// containerStart prepares the container to run by setting up everything the
// container needs, such as storage and networking, as well as links
// between containers. The container is left waiting for a signal to
// begin running.
func (daemon *Daemon) containerStart(container *container.Container, checkpoint string, checkpointDir string, resetRestartManager bool) (err error) {
	start := time.Now()
	container.Lock()
	defer container.Unlock()

	if resetRestartManager && container.Running { // skip this check if already in restarting step and resetRestartManager==false
		return nil
	}

	if container.RemovalInProgress || container.Dead {
		return errdefs.Conflict(errors.New("container is marked for removal and cannot be started"))
	}

	if checkpointDir != "" {
		// TODO(mlaventure): how would we support that?
		return errdefs.Forbidden(errors.New("custom checkpointdir is not supported"))
	}

	// if we encounter an error during start we need to ensure that any other
	// setup has been cleaned up properly
	defer func() {
		if err != nil {
			container.SetError(err)
			// if no one else has set it, make sure we don't leave it at zero
			if container.ExitCode() == 0 {
				container.SetExitCode(128)
			}
			if err := container.CheckpointTo(daemon.containersReplica); err != nil {
				logrus.Errorf("%s: failed saving state on start failure: %v", container.ID, err)
			}
			container.Reset(false)

			daemon.Cleanup(container)
			// if containers AutoRemove flag is set, remove it after clean up
			if container.HostConfig.AutoRemove {
				container.Unlock()
				if err := daemon.ContainerRm(container.ID, &types.ContainerRmConfig{ForceRemove: true, RemoveVolume: true}); err != nil {
					logrus.Errorf("can't remove container %s: %v", container.ID, err)
				}
				container.Lock()
			}
		}
	}()

	if err := daemon.conditionalMountOnStart(container); err != nil {
		return err
	}

	if err := daemon.initializeNetworking(container); err != nil {
		return err
	}

	spec, err := daemon.createSpec(container)
	if err != nil {
		return errdefs.System(err)
	}

	if resetRestartManager {
		container.ResetRestartManager(true)
		container.HasBeenManuallyStopped = false
	}

	if daemon.saveApparmorConfig(container); err != nil {
		return err
	}

	if checkpoint != "" {
		checkpointDir, err = getCheckpointDir(checkpointDir, checkpoint, container.Name, container.ID, container.CheckpointDir(), false)
		if err != nil {
			return err
		}
	}

	createOptions, err := daemon.getLibcontainerdCreateOptions(container)
	if err != nil {
		return err
	}

	ctx := context.TODO()

	err = daemon.containerd.Create(ctx, container.ID, spec, createOptions) // daemon/daemon.go#NewDaemon#L1043 libcontainerd.NewClient
	if err != nil {
		if errdefs.IsConflict(err) {
			logrus.WithError(err).WithField("container", container.ID).Error("Container not cleaned up from containerd from previous run")
			// best effort to clean up old container object
			daemon.containerd.DeleteTask(ctx, container.ID)
			if err := daemon.containerd.Delete(ctx, container.ID); err != nil && !errdefs.IsNotFound(err) {
				logrus.WithError(err).WithField("container", container.ID).Error("Error cleaning up stale containerd container object")
			}
			err = daemon.containerd.Create(ctx, container.ID, spec, createOptions)
		}
		if err != nil {
			return translateContainerdStartErr(container.Path, container.SetExitCode, err)
		}
	}

	// TODO(mlaventure): we need to specify checkpoint options here
	pid, err := daemon.containerd.Start(context.Background(), container.ID, checkpointDir,
		container.StreamConfig.Stdin() != nil || container.Config.Tty,
		container.InitializeStdio)
	if err != nil {
		if err := daemon.containerd.Delete(context.Background(), container.ID); err != nil {
			logrus.WithError(err).WithField("container", container.ID).
				Error("failed to delete failed start container")
		}
		return translateContainerdStartErr(container.Path, container.SetExitCode, err)
	}

	container.SetRunning(pid, true)
	container.HasBeenStartedBefore = true
	daemon.setStateCounter(container)

	daemon.initHealthMonitor(container)

	if err := container.CheckpointTo(daemon.containersReplica); err != nil {
		logrus.WithError(err).WithField("container", container.ID).
			Errorf("failed to store container")
	}

	daemon.LogContainerEvent(container, "start")
	containerActions.WithValues("start").UpdateSince(start)

	return nil
}
```

上边这段代码到 err = daemon.containerd.Create(ctx, container.ID, spec, createOptions) // daemon/daemon.go#NewDaemon#L1043 libcontainerd.NewClient 这里会比较麻烦，需要到`libcontainerd/libcontainerd_linux.go#NewClient`寻找源码

```go
// libcontainerd/remote/client.go#Creat(
func (c *client) Create(ctx context.Context, id string, ociSpec *specs.Spec, runtimeOptions interface{}) error {
	bdir := c.bundleDir(id)
	c.logger.WithField("bundle", bdir).WithField("root", ociSpec.Root.Path).Debug("bundle dir created")

	_, err := c.client.NewContainer(ctx, id,
		containerd.WithSpec(ociSpec),
		containerd.WithRuntime(runtimeName, runtimeOptions),
		WithBundle(bdir, ociSpec),
	)
	if err != nil {
		if containerderrors.IsAlreadyExists(err) {
			return errors.WithStack(errdefs.Conflict(errors.New("id already in use")))
		}
		return wrapError(err)
	}
	return nil
}
```

# 技巧

在这过程中，比较方便的寻找代码实现方法是 比如 ContainerExecStart 是一个接口中的方法，希望找到她的实现可以通过比较简单的搜索注释寻找到方法：比如搜索 `//ContainerExecStart`
