# tap-s3-csv

This is a [Singer](https://singer.io) tap that reads data from files located inside a given S3 bucket and produces JSON-formatted data following the [Singer spec](https://github.com/singer-io/getting-started/blob/master/SPEC.md).

## How to use it

`tap-s3-csv` works together with any other [Singer Target](https://singer.io) to move data from s3 to any target destination.

### Install

First, make sure Python 3 is installed on your system or follow these
installation instructions for [Mac](http://docs.python-guide.org/en/latest/starting/install3/osx/) or
[Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-python-3-and-set-up-a-local-programming-environment-on-ubuntu-16-04).

It's recommended to use a virtualenv:

```bash
 python3 -m venv ~/.virtualenvs/tap-s3-csv
 source ~/.virtualenvs/tap-s3-csv/bin/activate
 pip install -U pip setuptools
 pip install -e '.[dev]'
```

### Configuration

Here is an example of basic config, and a bit of a run down on each of the properties:

```
{
    "start_date": "2017-11-02T00:00:00Z",
    "bucket": "my-bucket",
    "tables": "[{\"search_prefix\":\"exports\",\"search_pattern\":\"my_table\\\\/.*\\\\.csv\",\"table_name\":\"my_table\",\"key_properties\":\"id\",\"date_overrides\":\"created_at\",\"delimiter\":\",\"}]",
}
```

- **start_date**: Required - This is the datetime that the tap will use to look for newly updated or created files, based on the modified timestamp of the file.
- **bucket**: Required - The name of the bucket to search for files under.
- **tables**: Optional, but not tested due to time pressue - An escaped JSON string that the tap will use to search for files, and emit records as "tables" from those files. Will be used by a [`voluptuous`](https://github.com/alecthomas/voluptuous)-based configuration checker.

The `table` field consists of one or more objects, JSON encoded as an array and escaped using backslashes (e.g., `\"` for `"` and `\\` for `\`), that describe how to find files and emit records. A more detailed (and unescaped) example below:

```
[
    {
        "search_prefix": "exports"
        "search_pattern": "my_table\\/.*\\.csv",
        "table_name": "my_table",
        "key_properties": "id",
        "date_overrides": "created_at",
        "delimiter": ","
    },
    ...
]
```

- **search_prefix**: This is a prefix to apply after the bucket, but before the file search pattern, to allow you to find files in "directories" below the bucket.
- **search_pattern**: This is an escaped regular expression that the tap will use to find files in the bucket + prefix. It's a bit strange, since this is an escaped string inside of an escaped string, any backslashes in the RegEx will need to be double-escaped.
- **table_name**: This value is a string of your choosing, and will be used to name the stream that records are emitted under for files matching content.
- **key_properties**: These are the "primary keys" of the CSV files, to be used by the target for deduplication and primary key definitions downstream in the destination.
- **date_overrides**: Specifies field names in the files that are supposed to be parsed as a datetime. The tap doesn't attempt to automatically determine if a field is a datetime, so this will make it explicit in the discovered schema.
- **delimiter**: This allows you to specify a custom delimiter, such as `\t` or `|`, if that applies to your files.

### Run

```
tap-s3-csv -c config.sample.json --catalog catalog.sample.json
```

---

Copyright &copy; 2018 Stitch
