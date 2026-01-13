# CMP-1445 — Customer captures or picks job photos with trade-specific guidance

As a customer standing in front of the job, I want to take or pick
up to 12 photos and be told which shots are useful for this kind of
work, so that the contractor and the AI can see what the job
involves.

Acceptance criteria:
- Before the first photo, guidance for the chosen trade names at
  least three shots to take (roofing: whole roof from the street,
  damage close up, attic if reachable).
- Camera capture and gallery pick both work in Safari, Chrome and
  the Facebook and Instagram in-app browsers; HEIC is accepted.
- Thumbnails appear as photos are added; any can be removed before
  submission; the 13th is refused with a message.
- The contractor's gallery shows the photos in the order added,
  correctly oriented.

## Implementation notes

Implementation: capture via <input capture="environment"> and
gallery pick via a plain file input, verified in Safari, Chrome and
the Facebook and Instagram in-app browsers on BrowserStack.
Compression in a Web Worker with createImageBitmap + canvas to
about 1.5 MB (quality stepped down to a floor of 0.6, then accept
up to 2 MB). HEIC decoded on device with heic2any, falling back to
a server conversion endpoint when the decoder throws. Orientation
EXIF applied before encoding. Guidance panel per trade from the
question-set config, shown before the first photo, naming at least
three shots. Thumbnails, remove, and a 12-item cap with the copy
team's message on the 13th.

Acceptance (execution-specific):
- A 6 MB 4032x3024 JPEG encodes to 1.5 MB or less in under 3 s on
  an iPhone 11 in the BrowserStack run (timing recorded in the PR).
- HEIC fixtures (portrait and landscape) render upright in the
  thumbnail and the stored file (Playwright + image diff).
- The 13th photo is refused and removing one allows another.
- The contractor gallery shows photos in the order added.
