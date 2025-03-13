# Django dumpdata encoding issue

A quick Django app to demonstrate a character encoding issue in `dumpdata` on Windows.

## Description

Using the management command `dumpdata` with an explicit `--output` argument raises an exception and fails to complete
when the database contains Unicode characters outside of those handled by Windows-1252.

The issue appears to be that while Django databases explicitly use UTF-8 encoding, the `dumpdata` management command
does not specify an encoding or an encoding error handler when opening output streams, allowing `open` to fall back on
`locale.getencoding()` (which on Windows is `cp1252`) and `'strict'` (which raises `UnicodeError`s).

## Steps to reproduce

0. Clone and set up this repository on a Windows device.
(Tested on `3.12.4 (tags/v3.12.4:8e8a4ba, Jun  6 2024, 19:30:16) [MSC v.1940 64 bit (AMD64)]`
and `3.13.2 (main, Feb 12 2025, 14:49:53) [MSC v.1942 64 bit (AMD64)]`)

    ```bat
    git clone https://github.com/tboz203/django-dumpdata-encoding-issue
    cd django-dumpdata-encoding-issue
    python -m venv .venv
    .venv\Scripts\activate
    python -m pip install -r requirements.txt
    python .\manage.py migrate
    ```

1. Load the included database fixture.

    ```bat
    python .\manage.py loaddata db.dump.jsonl
    ```

2. Dump the database to a file.

    ```bat
    python .\manage.py dumpdata --output output.json --traceback
    ```

This should produce output similar to the following:

```
Traceback (most recent call last):
  File "C:\Users\tbozeman\workspace\django-dumpdata-encoding-issue\manage.py", line 23, in <module>
    main()
  File "C:\Users\tbozeman\workspace\django-dumpdata-encoding-issue\manage.py", line 19, in main
    execute_from_command_line(sys.argv)
  File "C:\Users\tbozeman\workspace\django-dumpdata-encoding-issue\.venv\Lib\site-packages\django\core\management\__init__.py", line 442, in execute_from_command_line
    utility.execute()
  File "C:\Users\tbozeman\workspace\django-dumpdata-encoding-issue\.venv\Lib\site-packages\django\core\management\__init__.py", line 436, in execute
    self.fetch_command(subcommand).run_from_argv(self.argv)
  File "C:\Users\tbozeman\workspace\django-dumpdata-encoding-issue\.venv\Lib\site-packages\django\core\management\base.py", line 413, in run_from_argv
    self.execute(*args, **cmd_options)
  File "C:\Users\tbozeman\workspace\django-dumpdata-encoding-issue\.venv\Lib\site-packages\django\core\management\base.py", line 459, in execute
    output = self.handle(*args, **options)
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "C:\Users\tbozeman\workspace\django-dumpdata-encoding-issue\.venv\Lib\site-packages\django\core\management\commands\dumpdata.py", line 269, in handle
    serializers.serialize(
  File "C:\Users\tbozeman\workspace\django-dumpdata-encoding-issue\.venv\Lib\site-packages\django\core\serializers\__init__.py", line 134, in serialize
    s.serialize(queryset, **options)
  File "C:\Users\tbozeman\workspace\django-dumpdata-encoding-issue\.venv\Lib\site-packages\django\core\serializers\base.py", line 145, in serialize
    self.end_object(obj)
  File "C:\Users\tbozeman\workspace\django-dumpdata-encoding-issue\.venv\Lib\site-packages\django\core\serializers\json.py", line 54, in end_object
    json.dump(self.get_dump_object(obj), self.stream, **self.json_kwargs)
  File "C:\Users\tbozeman\AppData\Local\Programs\Python\Python312\Lib\json\__init__.py", line 180, in dump
    fp.write(chunk)
  File "C:\Users\tbozeman\AppData\Local\Programs\Python\Python312\Lib\encodings\cp1252.py", line 19, in encode
    return codecs.charmap_encode(input,self.errors,encoding_table)[0]
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
UnicodeEncodeError: 'charmap' codec can't encode character '\u2757' in position 13: character maps to <undefined>
Exception ignored in: <generator object cursor_iter at 0x000002DBAEFE0AE0>
Traceback (most recent call last):
  File "C:\Users\tbozeman\workspace\django-dumpdata-encoding-issue\.venv\Lib\site-packages\django\db\models\sql\compiler.py", line 2115, in cursor_iter
    cursor.close()
sqlite3.ProgrammingError: Cannot operate on a closed database.
```
