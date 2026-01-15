# CMP-1448 — Upload survives a poor connection and strips location data

As a customer on a weak mobile connection, I want my photos to
finish uploading even if the signal drops, and I do not want the
photo's GPS location stored, so that my request goes through and my
privacy is respected.

Acceptance criteria:
- Photos are compressed on the device to about 1.5 MB before upload
  without blocking the form.
- A connection lost for 30 seconds mid-upload resumes automatically
  and the photo arrives intact (checksum match).
- GPS EXIF data is removed before storage; orientation EXIF is
  applied and then removed.
- A photo that fails the virus scan is rejected with a message; the
  others are kept.

## Implementation notes

Implementation: a tus 1.0 endpoint at /api/intake/uploads backed by
the bucket for the contractor's region, server-side encryption with
the region's KMS key; tus-js-client on the device with 1 MB chunks
and retry with backoff. On completion the object is re-encoded
server-side with all EXIF dropped (GPS included), then scanned
synchronously by Security's scanner in its new request/response
mode; a reject deletes the object and returns the reason to the
client, which keeps the other photos. Photos attach to the request
on submission; a 90-day lifecycle rule deletes them after the
request closes; the gallery serves 15-minute signed URLs.

Acceptance (execution-specific):
- Chaos test: the proxy drops connections for 30 s mid-transfer;
  the client resumes and the stored SHA-256 matches the source for
  50 of 50 runs.
- Unit test with exif-reader on a fixture that had GPS tags: none
  present in the stored object.
- An EICAR file is rejected with reason "virus" and no object
  remains.
- A CA contractor's upload lands in the CA bucket only; the EU
  bucket is unreachable from the US service role.
