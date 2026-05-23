This is an updated version of the Google EI parsing script from 4n6-scripts created by http://cheeky4n6monkey.blogspot.com/

TO USE:

1. Open up command prompt
2. cd into the directory containing the script
3. Type in ```python google-ei-time.py -u "```
4. Paste in your Google URL
5. Add another quotation mark at the end
6. Press enter and wait for results


Example:
```
F:\Projects\Google_EI_Parser>python google-ei-time.py -u "https://www.google.com/search?q=hello+world&sca_esv=70a532a702b7bdb6&source=hp&ei=_7_WaMrqH-n8p84P8uaYqQs&iflsig=AOw8s4IAAAAAaNbODx_z1t4CNs7GPOQ9nHozWG0dUacj"
Running google-ei-time.py v2025-09-26 (Python 3)

URL's ei term = _7_WaMrqH-n8p84P8uaYqQs
Padded base64 string = _7_WaMrqH-n8p84P8uaYqQs=
Extracted timestamp = 1758904319
F:\Projects\Google_EI_Parser\google-ei-time.py:123: DeprecationWarning: datetime.datetime.utcfromtimestamp() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.fromtimestamp(timestamp, datetime.UTC).
  datetimestr = datetime.datetime.utcfromtimestamp(timestamp).strftime('%Y-%m-%dT%H:%M:%S')
Human readable timestamp (UTC) = 2025-09-26T16:31:59
```


Compile with:
```python -m nuitka --standalone --onefile google-ei-time.py```