# Getting Started

Thanks for using Search Lib. The goal of this document is to get you up and running as fast as possible. One of the primary goals for Search Lib is that it takes no time to get up and running.

## Terms
- **Distance Metric**: Is a function that computes the distance between two strings. You can measure these distances in many different ways. One example is the Levenshtein distance, which is the number of insertions, deletions, and substitutions required to transform one string into another. In this project the IDistanceMetric interface represents a computation of a metric. An object that implements IDistanceMetric represents one computation and is stateful. Make sure to reset the state before reusing.
- **Search Method**: Is a strategy for sorting a set of strings in order of distance from a query string. The ISearchMethod interface encapsulates this strategy. All ISearchMethods are offline and require initializing with a set of strings before use. This allows optimizations and search indexing which speeds up search times. The simplest of these classes is LinearSearch which just runs an IDistanceMetric over every string in the set and sorts the results.
- **Search Provider**: Is simply an interface with a search method. Mostly these are wrapper classes to make them compatible with EditorTime or Runtime. They also manage multithreading of the search. ISearchProvider will be synchronous and blocking, while anything implementing INonBlockingSearchProvider does its computation in a worker thread.

## A Basic Setup
 To start you'll need to decide on an IDistanceMetric, ISearchMethod, and an ISearchProvider. This requires a technical overview, so for now I'll choose for you! Get ready for this mouthful of code. 
```c#
ISearchProvider provider = new EditorSearchProvider<LinearSearch<WagnerFischerDamerauLevenshtein>>();
```
This creates a search provider for the editor, which uses a LinearSearch with WagnerFischerDamerauLevenshtein as the distance metric. Our focus now is to get you up and running.

Next we need to make some settings.
```c#
SearchMethodSettings settings = new SearchMethodSettings() 
{
    MaxDistance = int.MaxValue, // Sets the maximum distance to be included in the search results
};
```

Then we need to provide our search provider with a list of options to relay to the search method. This can be any IEnumerable<string>. If you want to tie data to your strings, you can store everything in a dictionary.
```c#
Dictionary<string, Weapon> data = new Dictionary<string, Weapon>();
```

And finally we need to initialize the search provider.
```c#
provider.Initialize(data, settings);
```

Now we can search whatever we want!
```c#
List<SearchResult> results = provider.Search("sword");
```

The `SearchResult` struct contains the string and the distance from the query string. To get back the original Weapon object we can query the dictionary.
```c#
Weapon weapon = data[results[0].Option];
```

And that's it! The flow for INonBlockingSearchProvider is slightly different. Instead of returning a List<SearchResult> we get nothing, but you instead register for a callback on the provider.

```c#
provider.OnResultsReceived += (results) => {
    // Do something with the results
};
```

And an example for using monobehaviours at runtime can be found in the Examples folder. The API is similar, but adapted for monobehaviours.

## Contact
If you need any help feel free to reach out to [predashgames@gmail.com](mailto:predashgames@gmail.com)
