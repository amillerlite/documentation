---
title: Propagating C++ Trace Context
kind: documentation
code_lang: cpp
type: multi-code-lang
code_lang_weight: 50
---

## Overview

Datadog APM tracer supports [B3 headers extraction][11] and injection for distributed tracing.

Distributed headers injection and extraction is controlled by configuring injection/extraction styles. The supported styles for C++ are:

- Datadog: `Datadog`
- B3: `B3`

Injection styles can be configured using:

- Environment Variable: `DD_TRACE_PROPAGATION_STYLE_INJECT="Datadog B3"`

The value of the environment variable is a comma (or space) separated list of header styles that are enabled for injection. By default only Datadog injection style is enabled.

Extraction styles can be configured using:

- Environment Variable: `DD_TRACE_PROPAGATION_STYLE_EXTRACT="Datadog B3"`

The value of the environment variable is a comma (or space) separated list of header styles that are enabled for extraction. By default only Datadog extraction style is enabled.

If multiple extraction styles are enabled, the extraction attempt is done on the order those styles are configured and first successful extracted value is used.

### Inject and extract context for distributed tracing

Distributed tracing can be accomplished by [using the `Inject` and `Extract` methods on the tracer][9], which accept [generic `Reader` and `Writer` types][10]. Priority sampling (enabled by default) ensures uniform delivery of spans.

```cpp
// Allows writing propagation headers to a simple map<string, string>.
// Copied from https://github.com/opentracing/opentracing-cpp/blob/master/mocktracer/test/propagation_test.cpp
struct HTTPHeadersCarrier : HTTPHeadersReader, HTTPHeadersWriter {
  HTTPHeadersCarrier(std::unordered_map<std::string, std::string>& text_map_)
      : text_map(text_map_) {}

  expected<void> Set(string_view key, string_view value) const override {
    text_map[key] = value;
    return {};
  }

  expected<void> ForeachKey(
      std::function<expected<void>(string_view key, string_view value)> f)
      const override {
    for (const auto& key_value : text_map) {
      auto result = f(key_value.first, key_value.second);
      if (!result) return result;
    }
    return {};
  }

  std::unordered_map<std::string, std::string>& text_map;
};

void example() {
  auto tracer = ...
  std::unordered_map<std::string, std::string> headers;
  HTTPHeadersCarrier carrier(headers);

  auto span = tracer->StartSpan("operation_name");
  tracer->Inject(span->context(), carrier);
  // `headers` now populated with the headers needed to propagate the span.
}
```


[9]: https://github.com/opentracing/opentracing-cpp/#inject-span-context-into-a-textmapwriter
[10]: https://github.com/opentracing/opentracing-cpp/blob/master/include/opentracing/propagation.h
[11]: https://github.com/openzipkin/b3-propagation
