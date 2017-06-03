# FileSets

We's seen that you can explicitly list all the files in the EXTRACT statement. You may not want to list all the files manually because there are a lot them.

FileSets make it easy to define a pattern to identify a set of files to read.

In the simplest case let's get all the files in a folder.

```
@rs =
    EXTRACT 
        user   string,
        id     string,
    FROM 
        "/input/{*}"
    USING Extractors.Csv();
```

### Getting information about the filenames into the RowSet

Because we are reading rows from multiple files. it is convenient to for the rows to have some information about the filename it came from. We can adjust the query slightly to make this possible.

```
@rs =
    EXTRACT 
        user       string,
        id         string,
        __filename string
    FROM 
        "/input/{__filename}"
    USING Extractors.Csv();
```

You are probably wondering about the `__` in the column `__filename`. It isn't necessary at all, however it is useful as a way of marking that this information came from the process of extracting the file, not from the data in the file itself.

### Specifying a list of files that have an extension

This is very simple modification syntax. The example uses will extract all the files that end with ".csv".

```
@rs =
    EXTRACT 
        user   string,
        id     string,
    FROM 
        "/input/{*}.csv"
    USING Extractors.Csv();
```

## Using WHERE to filter the files.

Suppose you have a folder with 5000 files that are names as "data1".csv, "data2.csv", ..., "data4999.csv", "data5000.csv". Often you want the only a certain range of those files for example only that match  7 to 21.

First, lets see how to match the basic pattern of "data + num + .csv"

```
@rs =
    EXTRACT 
        user       string,
        id         string
    FROM 
        "/input/data{*}.csv"
    USING Extractors.Csv();
```

Now to get the file number into the RowSet.

```
@rs =
    EXTRACT 
        user       string,
        id         string,
        __filenum  int,
    FROM 
        "/input/data{__filenum}.csv"
    USING Extractors.Csv();
```

Right now the script will still attempt to read all 5000 files. But because we have added the \_\_filenum column from the extraction creating future rowsets can use that information to avoid reading uneccessary files.

```
@rs =
    EXTRACT 
        user       string,
        id         string,
        __filenum  int,
    FROM 
        "/input/data{__filenum}.csv"
    USING Extractors.Csv();

@rs =
    SELECT *
    FROM @rs
    WHERE 
        ( __filenum >= 7 ) 
        AND (__filenum <= 21);
```

## FileSets with dates and date ranges

File names or paths often include information about dates and times. This is often the case for log files. FileSets make it easy to handle these cases.

Image we have files that are named in this pattern "data-YEAR-MONTH-DAY.csv". The following query reads all the files where with that pattern and filters them to filenames matching a date range

```
@rs = 
  EXTRACT 
      user    string,
      id      string,
      __date  DateTime
  FROM 
    "/input/data-{__date:yyyy}-{__date:MM}-{__date:dd}.csv"
  USING Extractors.Csv();

@rs = 
  SELECT * FROM @rs
  WHERE date >= System.DateTime.Parse("2016/1/1") AND
        date < System.DateTime.Parse("2016/2/1");
```

Now what if the files are also placed in a folder hierarchy with one folder for year, one for month, and one for day.

A file path would look like "/input/2017/03/31/data-2017-03-31.csv"

```
@rs =
    EXTRACT 
      user    string,
      id      string,
      __date  DateTime
  FROM 
    "/input/{__date:yyyy}/{__date:MM}/{__date:dd}/data-{__date:yyyy}-{__date:MM}-{__date:dd}.csv"
  USING Extractors.Csv();

@rs = 
  SELECT * FROM @rs
  WHERE date >= System.DateTime.Parse("2016/1/1") AND
        date < System.DateTime.Parse("2016/2/1");
```


