---
title: "Opentracing - jaeger"
author: "Exfly"
cover: "/media/img/icon/logo43.svg"
tags: []
date: 2020-04-24T08:41:08+08:00
---

文章简介：介绍 Opentracing - jaeger

<!--more-->

![jaeger](/media/img/opentracing/jaeger.png)

## agent

Agent 处于 jaeger-client 和 collector 之间，属于代理的作用，主要是把 client 发送过来的数据从 thrift 转为 Batch，并通过 RPC 批量提交到 collector

> [jaegertracing/jaeger/cmd/agent/app/flags.go#L62](https://github.com/jaegertracing/jaeger/blob/886b96574253a005ee7ebe74140098f3fe183606/cmd/agent/app/flags.go#L62)

```go
var defaultProcessors = []struct {
	model    Model
	protocol Protocol
	port     int
}{
	{model: "zipkin", protocol: "compact", port: 5775},
	{model: "jaeger", protocol: "compact", port: 6831},
	{model: "jaeger", protocol: "binary", port: 6832},
}
```

> [jaegertracing/jaeger/cmd/agent/app/servers/tbuffered_server.go#L82](https://github.com/jaegertracing/jaeger/blob/b4670412977f653dfbb2671f7f04756a30e897e6/cmd/agent/app/servers/tbuffered_server.go#L82)

```go
// Serve initiates the readers and starts serving traffic
func (s *TBufferedServer) Serve() {
	atomic.StoreUint32(&s.serving, 1)
	for s.IsServing() {
		readBuf := s.readBufPool.Get().(*ReadBuf)
		n, err := s.transport.Read(readBuf.bytes)
		if err == nil {
			readBuf.n = n
			s.metrics.PacketSize.Update(int64(n))
			select {
			case s.dataChan <- readBuf:
				s.metrics.PacketsProcessed.Inc(1)
				s.updateQueueSize(1)
			default:
				s.metrics.PacketsDropped.Inc(1)
			}
		} else {
			s.metrics.ReadError.Inc(1)
		}
	}
}
```

> [jaegertracing/jaeger/blob/master/cmd/agent/app/processors/thrift_processor.go#L114](https://github.com/jaegertracing/jaeger/blob/ae86232300d47061eeeed6715004d2c8e889dcf0/cmd/agent/app/processors/thrift_processor.go#L114)

```go
// processBuffer reads data off the channel and puts it into a custom transport for
// the processor to process
func (s *ThriftProcessor) processBuffer() {
	for readBuf := range s.server.DataChan() {
		protocol := s.protocolPool.Get().(thrift.TProtocol)
		payload := readBuf.GetBytes()
		protocol.Transport().Write(payload)
		s.logger.Debug("Span(s) received by the agent", zap.Int("bytes-received", len(payload)))

		if ok, err := s.handler.Process(protocol, protocol); !ok {
			s.logger.Error("Processor failed", zap.Error(err))
			s.metrics.HandlerProcessError.Inc(1)
		}
		s.protocolPool.Put(protocol)
		s.server.DataRecd(readBuf) // acknowledge receipt and release the buffer
	}
}
```

> [jaegertracing/jaeger/thrift-gen/agent/agent.go#L187](https://github.com/jaegertracing/jaeger/blob/43be2e7b6be62b04bb40ac564a4be8f5cb7cf607/thrift-gen/agent/agent.go#L187)

```go
func (p *agentProcessorEmitBatch) Process(seqId int32, iprot, oprot thrift.TProtocol) (success bool, err thrift.TException) {
	args := AgentEmitBatchArgs{}
	if err = args.Read(iprot); err != nil {
		iprot.ReadMessageEnd()
		return false, err
	}

	iprot.ReadMessageEnd()
	var err2 error
	if err2 = p.handler.EmitBatch(args.Batch); err2 != nil {
		return true, err2
	}
	return true, nil
}
```

> [jaegertracing/jaeger/thrift-gen/jaeger/tchan-jaeger.go#L39](https://github.com/jaegertracing/jaeger/blob/43be2e7b6be62b04bb40ac564a4be8f5cb7cf607/thrift-gen/jaeger/tchan-jaeger.go#L39)

## Collector

### 接收 Agent 的数据

> [jaegertracing/jaeger/cmd/collector/app/handler/thrift_span_handler.go#L60](https://github.com/jaegertracing/jaeger/blob/3ae21efe69cf5657b9b39a873edc0bcc85b84407/cmd/collector/app/handler/thrift_span_handler.go#L60)

> [比较舒服的维护metrics的场景](https://github.com/jaegertracing/jaeger/blob/5fb7d7295bf99210ae9c8f6364da5356e61afefb/cmd/collector/app/span_processor.go#L78)

## references

- [jaeger](https://github.com/jukylin/blog/blob/master/Jaeger%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E2%80%94%E2%80%94%E7%AA%A5%E8%A7%86%E5%88%86%E5%B8%83%E5%BC%8F%E7%B3%BB%E7%BB%9F%E5%AE%9E%E7%8E%B0.md)
