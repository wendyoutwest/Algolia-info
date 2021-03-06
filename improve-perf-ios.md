# Improve performance

Mitigate the impact of a slow network in your search application.

## Debouncing

Debouncing is the process of ignoring overly frequent events, keeping only the last one in a series of adjacent events.

The `Debouncer` class provides a generic way of debouncing calls. It is useful in order to avoid triggering too many search requests, for example, when a UI widget is continuously firing updates (e.g. a slider).

## Throttling

Throttling works in a similar fashion to debouncing, yet it ensures a constant throughput.

The `Throttler` class delays calls for a given amount of time before they are fired. The throttler fires the latest call at regular intervals. No matter how many calls are made, exactly one call per interval is fired.

## Request strategy

By default, a Searcher launches a request every time you call `search(...)`.

When network conditions are bad, for example with high latency, poor bandwidth, or packet loss, the network may not be able to cope with as-you-type search, which leads to a poor user experience.

The `Searcher` class accepts an optional **strategy delegate** that takes care of deciding how to perform searches. This delegate can decide to drop requests, throttle them, or even alter their metadata.

### Adaptive network

The library provides one request strategy implementation, **AdaptiveNetworkStrategy**. This strategy monitors the response time of every request, and based on the observed response times, switches between various modes: **realtime**, **throttled** and **manual**.

**Realtime mode** is the default, and all requests are fired immediately. This is best for an as-you-type search context and provides the optimal user experience when network conditions are good.

**Throttle mode** is best when the network starts to degrade. Throttle mode delays requests, dropping them along the way, to ensure a maximum throughput. The throttling delay is dynamically adjusted so that the search throughput matches the current network's capabilities as closely as possible. Throttle mode avoids slow response time and avoids requests stacking up inside the pipeline. 

**Manual mode** is necessary when the network is extremely slow. It's better to disable as-you-type and inform the users that they need to explicitly submit their searches. In manual mode, only final searches display.

The **adaptive network** strategy requires monitoring response times using a `ResponseTimeStats`, an `AdaptiveNetworkStrategy`, and a `Searcher`.

```swift
let searcher = /* your searcher */
let stats = ResponseTimeStats(searcher: searcher)
let strategy = AdaptiveNetworkStrategy(stats: stats)
searcher.strategy = strategy
```

**Note:** A `Searcher` does not retain its strategy. The lifetime must exceed that of the searcher.


## Writing your own strategy

Implement the `RequestStrategy` protocol, which contains one method, `performSearch(from:userInfo:with:)`. When the `search(...)` method is invoked, the Searcher calls the strategy and provides the search metadata via the `userInfo` parameter.


## Optimize build size

In some cases it is better to use the core parts of InstantSearch and download only specific parts of the library.

### CocoaPods

```ruby
pod 'InstantSearch', '~> 2.0'
# pod 'InstantSearch/Widgets' for access to everything
# pod 'InstantSearch/Core' for access to everything except the UI widgets
# pod 'InstantSearch/Client' for access only to the API Client
```

### Carthage

```ruby
github "algolia/instantsearch-ios" ~> 2.0 # for access to everything
# github "algolia/instantsearch-core-swift" ~> 3.0 # for access to everything except the UI widgets
# github "algolia/algoliasearch-client-swift" ~> 5.0 # for access only to the API Client
```

## Caching

Enable the search cache to easily cache the results of the search queries. The results are cached for a defined amount of time with a default of two minutes. Simulate a pre-caching mechanism by making a preemptive search query.

Cache is disabled by default.

```swift
// Enable the search cache with default settings.
InstantSearch.shared.searchCacheEnabled = true

// Enable the search cache with a TTL of 5 minutes.
InstantSearch.shared.searchCacheEnabled = true
InstantSearch.shared.searchCacheExpiringTimeInterval = 300
```

## Queries per second (QPS)

[Search operations](<%= app_data.instantsearch.links.faq.operations %>) are limited by the [maximum QPS](<%= app_data.instantsearch.links.faq.qps %>) (the number of queries performed per second per the plan).

Every key that you press in InstantSearch counts as one operation. Depending on the widgets in your search interface, you might have more operations counted for each keystroke. For example, if you have a search consisting of a SearchBox, a Menu, and a RefinementList,  every keystroke triggers one operation. When you refine the Menu or RefinementList, it triggers a second operation for each keystroke.

Most search interfaces using InstantSearch trigger one operation per keystroke. Every refined widget (clicked widget) adds one operation to the total count.

If issues with the QPS arise, consider implementing a debounced [`SearchBox`](/doc/api-reference/widgets/search-box/ios/).
