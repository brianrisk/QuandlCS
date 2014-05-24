## QuandlCS - TypedDatasets Branch

***

QuandlCS is a simple wrapper around the Quandl API to provide easy access. The library currently provides download functionality supporting simple, multiset, favourite, metadata and search request downloads. 

Started as a project in an attempt at my colleagues [Project365](http://staticcast.wordpress.com/2013/12/28/programming-365/)  

Visit the [online reference](http://www.quandl.com/help/api) for more information about the Quandl API.

This branch is an experimental feature to allow for typed datasets. It was originally based on something I had been working on for a potential Upload API but was shelved as the Upload API was taken offline. 

I would not recommend using this in anything important. The error handling is non existent and there are a few bugs that I know of and probably many more that I don't.


### Tutorial
The CS API is simple to use and provides type safety garuantees and basic validation on data before requesting. Interaction is based around the concept of a request with all download requests implementing a single interface. Once a request is created this can be used with the QuandlConnection object to download and return the data or to generate the request string URL (`www.quandl.com/api/v1/datasets/PRAGUESE/PX.json`) to handle manually. 

#### Types
The API has a few fundamental types that mimick Quandl variables and options. 

###### Datacode
The datacode class represents the Quandl Code that uniquely identifies each dataset. It is made up of a Source and a Code. 

```c#
Datacode code = new Datacode("PRAGUESE", "PX"); // PRAGUESE is the source, PX is the datacode
```

When used this class will validate that the source and code provided do not contain any invalid characters but will not validate that the code does exist.


###### FileFormats
The FileFormats enum represents each of the available [output formats](http://www.quandl.com/help/api#Basic+Principles) from the Quandl API. Not all of the formats will be valid for each of the different requests. (e.g. Metadata requests are only available in XML and JSON formats... this is validated by the QuandlMetadataRequest class when used)


###### Frequencies
The Frequencies enum is used to specify which [frequency collapse](http://www.quandl.com/help/api#Frequency+Collapsing) is required for the dataset being downloaded.


###### SortOrders
The SortOrders enum, as you can probably imagine, specified if the data should be sorted ascending or descending. 


###### Transformations
The Transformations enum specifies which [transformation](http://www.quandl.com/help/api#Transformations) should be applied to the before downloading. 


#### Download
The simplest and most useful request to be made is a download. 

```c#
using System;
using QuandlCS.Requests;
using QuandlCS.Types;

namespace QuandlCSTest
{
  class Program
  {
    static void Main(string[] args)
    {
      QuandlDownloadRequest request = new QuandlDownloadRequest();
      request.APIKey = "1234-FAKE-KEY-4321";
      request.Datacode = new Datacode("PRAGUESE", "PX"); // PRAGUESE is the source, PX is the datacode
      request.Format = FileFormats.JSON;
      request.Frequency = Frequencies.Monthly;
      request.Truncation = 150;
      request.Sort = SortOrders.Ascending;
      request.Transformation = Transformations.Difference;

      Console.WriteLine("The request string is : {0}", request.ToRequestString());
    }
  }
}
```

#### Multiset Download
The multiset download provides an almost identical interface to the simple download documented above, but instead of taking just one datacode object it provides two methods to add columns for the multiset. The columns are downloaded in the same order as they are added to this object. 

```c#
using System;
using QuandlCS.Requests;
using QuandlCS.Types;

namespace QuandlCSTest
{
  class Program
  {
    static void Main(string[] args)
    {
      QuandlMultisetRequest request = new QuandlMultisetRequest();
      request.APIKey = "1234-FAKE-KEY-4321";

      Datacode first = new Datacode("GOOG", "NASDAQ_GOOG");
      request.AddColumns(first);

      Datacode second = new Datacode("GOOG", "NASDAQ_AAPL");
      request.AddColumn(second, 4);
      
      request.Format = FileFormats.HTML;
      request.Frequency = Frequencies.Daily;
      request.Transformation = Transformations.None;
      request.StartDate = new DateTime(2000, 01, 01); // Terrible date to start as Google IPO was 19/08/2004...
      request.EndDate = new DateTime(2014, 01, 19);

      Console.WriteLine("The request string is : {0}", request.ToRequestString());
    }
  }
}
```

#### Metadata Download
The metadata download is useful for gathering the key details about a dataset. It provides a very simple interface needing only a dataset, format and an API key.  

```c#
using System;
using QuandlCS.Requests;
using QuandlCS.Types;

namespace QuandlCSTest
{
  class Program
  {
    static void Main(string[] args)
    {
      QuandlMetadataRequest request = new QuandlMetadataRequest();
      request.APIKey = "1234-FAKE-KEY-4321";
      request.Datacode = new Datacode("NSE", "OIL");      
      request.Format = FileFormats.XML;

      Console.WriteLine("The request string is : {0}", request.ToRequestString());
    }
  }
}
```

#### Favourites Download
The favourites download is used to gather information about the current user (as specified through the required API key) and which datasets are their favourites. I imagine this would be useful if you were to write a desktop application to allow users to manage their Quandl data... that's what I am planning to use it for.

```c#
using System;
using QuandlCS.Requests;
using QuandlCS.Types;

namespace QuandlCSTest
{
  class Program
  {
    static void Main(string[] args)
    {
      QuandlFavouritesRequest request = new QuandlFavouritesRequest();
      request.APIKey = "1234-FAKE-KEY-4321";
      request.Format = FileFormats.XML;

      Console.WriteLine("The request string is : {0}", request.ToRequestString());
    }
  }
}
```

#### Search
The search is pretty self explanatory in terms of what it is for... 

```c#
using System;
using QuandlCS.Requests;
using QuandlCS.Types;

namespace QuandlCSTest
{
  class Program
  {
    static void Main(string[] args)
    {
      QuandlSearchRequest request = new QuandlSearchRequest();
      request.APIKey = "1234-FAKE-KEY-4321";
      request.Format = FileFormats.XML;
      request.SearchQuery = "OIL";

      Console.WriteLine("The request string is : {0}", request.ToRequestString());
    }
  }
}
```

#### QuandlConnection
In order to make using the request objects easier there is also a QuandlConnection class that will take in any of the requests and then download the data returning it as a string to the caller...


```c#
using System;
using System.IO;
using QuandlCS.Connection;
using QuandlCS.Interfaces;
using QuandlCS.Requests;
using QuandlCS.Types;

namespace QuandlCSTest
{
  class Program
  {
    static void Main(string[] args)
    {
      IQuandlRequest request = CreateRequest(); // Implementation of CreateRequest() left as a challenge to the reader

      IQuandlConnection connection = new QuandlConnection();
      string data = connection.Request(request);
      using (StreamWriter writer = new StreamWriter(@"C:\TestOutput.xml"))
      {
        writer.Write(data);
      }     
    }
  }
}
```

#### TypedDatasets
This is an experimental feature to allow for type safety of downloaded data. Use with caution.

The idea behind how this works is that the user of API will create a new class to represent the data they will be downloading. As part of this they will apply special attributes to the class so that we can link the downloaded unsafe data to the typed object. 


###### DatasetAttribute
The Dataset attribute contains the details of the Quandl dataset the class is associated with. Currently this is only required if using the DatasetDownloader. 

###### ColumnAttribute
The Column attribute contains the details of the column that this property matches to. This is required by both the DatsetDownloader and the ColumnExtractor. 

This is based on the assumption that each dataset will have unique names for each of the columns... I am not sure if that is true so this is one of the things that may change in the future.



Below is an example dataset class 


```c#
using System;
using QuandlCS.DataTyping;

namespace QuandlTypedData
{
  [Dataset("USD vs AUD", "QUANDL", "USDAUD")]
  public class USDAUD
  {
    [Column("Date")]
    public DateTime Date
    {
      get;
      set;
    }

    [Column("Rate")]
    public double Rate
    {
      get;
      set;
    }

    [Column("High (est)")]
    public double High
    {
      get;
      set;
    }

    [Column("Low (est)")]
    public double Low
    {
      get;
      set;
    }
  }
}
```





:koala: