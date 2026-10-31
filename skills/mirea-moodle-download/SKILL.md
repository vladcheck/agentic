---
name: mirea-moodle-download
description: Use for Moodle pluginfile.php downloads using MoodleSession, including resume/PDF checks. Not for public files, credentials, or administration.
---

# Authenticated Moodle File Download

Download authentication-protected `pluginfile.php` resources from `<moodle-host>` without mistaking login HTML or truncated data for the requested file.

## Procedure

1. Ask the user for a fresh `MoodleSession` cookie value from browser DevTools: Application → Cookies. Treat the value as a secret and do not persist it in files or logs. If requests redirect with HTTP 303 to `/login/index.php`, ask for a fresh value.
2. Download with HTTP/1.1:

   ```bash
   curl -sS --http1.1 -b "MoodleSession=<moodle-session>" -o "<out.pdf>" "<pluginfile-url>"
   ```

   URL-encode non-ASCII path segments, including Cyrillic text. Keep `?forcedownload=1` in the URL.
3. If the server cuts the connection early, resume the same output file. Symptoms include `curl: (18)`, HTTP/2 `INTERNAL_ERROR`, `pdfinfo` reporting `Invalid XRef entry`, or `Couldn't read xref table`.

   ```bash
   for i in $(seq 1 8); do
     curl -sS --http1.1 -C - -b "MoodleSession=<moodle-session>" -o "<out.pdf>" "<pluginfile-url>" -w 'HTTP:%{http_code} bytes:%{size_download}\n'
     sleep 1
   done
   ```

   Stop when a pass reports `HTTP:200` and `bytes:0`. If eight passes do not complete the file, report the incomplete transfer instead of treating it as valid.
4. Verify the downloaded file:

   ```bash
   pdfinfo "<out.pdf>"
   pdftotext -l 1 -q "<out.pdf>" - | head -c 600
   file "<out.pdf>"
   ```

   Require `pdfinfo` to exit 0 with a plausible page count, the text sample to be readable, and `file` to identify a PDF document rather than HTML. HTML output, often approximately 39 KB and identified as `HTML document text`, indicates an expired session; obtain a fresh cookie and repeat the download.
