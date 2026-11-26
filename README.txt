Voight-Kampff data: every bot request to motdang.net and hongdam.net

Days: 2026-09-08 to 2026-11-25 (79 days, 14,079,491 rows). One Chiang Mai day (UTC+7) a file. A new day lands each night.

Releases here: one Parquet of bot rows and one CSV of people counts for each month, from the first day of the next month.
Daily files: https://motdang.net/voight-kampff/data/
  index.json              every day, its files, sizes, rows, sha256
  bots/<day>.parquet      one row per request, to the second (zstd Parquet)
  bots/<day>.csv.gz       the same rows as gzipped CSV
  people/<day>.csv.gz     browsers the classifier counts as people, as counts per hour, site, page and country
  fields.json             the fields below, as JSON

Who is in bots/: every row the site's classifier (publish/classify.js in github.com/motdang/mot-dang) does not call a browser,
plus browsers on data-centre networks, which it calls 'other'. People appear only as counts, in people/.

The log is Cloudflare Workers Analytics Engine, written by the site's own Worker on every request. It stores 90 days;
these files carry them past that. When the site is busy, Analytics Engine stores one row in N and says so in `weight`:
requests = SUM(weight * counted).

Network, data centre, TLS fingerprints and round-trip time start on 26 Nov 2026; earlier rows leave them empty.
No addresses are logged.

Fields, bots/:
  time             TIMESTAMP when the Worker saw the request, UTC, to the second
  site             VARCHAR   host asked for: motdang.net, hongdam.net, www., side ports (motdang.net:8443)
  kind             VARCHAR   the classifier's call (publish/classify.js): ai, search, seo, social, monitor, tool, other (unnamed or a data-centre network), none (no user-agent), ours
  caller           VARCHAR   the name the classifier gives it: GPTBot, meta-externalagent, Go-http-client, or the browser it claims to be
  path             VARCHAR   page or file asked for, no query string, cut at 96 characters
  country          VARCHAR   Cloudflare's country for the caller's address (XX = none, T1 = Tor)
  status           INTEGER   HTTP status the site answered with
  referrer         VARCHAR   host of the Referer header; (same site) when it was motdang.net itself
  method           VARCHAR   GET, HEAD, POST… or SHARE / FIND on the event rows below
  row              VARCHAR   request (an arrival), or an event the site wrote beside one: share, find, crawlpay, map, event
  f9               VARCHAR   request rows: 'search' on /api/v1/search · crawlpay rows: 'crawlpay' · share/find rows: the event
  f10              VARCHAR   search: the map cell searched (lat,lng to 2 places) · crawlpay: which arm · share codes left out
  f11              VARCHAR   search: the words, when they found nothing · crawlpay: the price it offered · share: channel
  f12              VARCHAR   crawlpay: its signature agent · share: page · find: what was judged
  f13              VARCHAR   event rows only
  f14              VARCHAR   event rows only
  asn              INTEGER   the network's number (autonomous system), from 26 Nov 2026
  network          VARCHAR   who owns that network, as registered, from 26 Nov 2026
  colo             VARCHAR   the Cloudflare data centre that answered, by airport code, from 26 Nov 2026
  connection       VARCHAR   HTTP version, TLS version and cipher, from 26 Nov 2026
  ciphers_sha1     VARCHAR   SHA-1 of the cipher list the caller offered: a fingerprint of its software, from 26 Nov 2026
  extensions_sha1  VARCHAR   SHA-1 of the TLS extensions it sent, in order; Chrome shuffles these on every connection, from 26 Nov 2026
  worker_ms        DOUBLE    time the site's Worker took, ms (request rows)
  d4               DOUBLE    search: map box width in degrees · find: rank · varies by row
  d5               DOUBLE    search: results · varies by row
  rtt_ms           DOUBLE    round trip between the caller and that data centre, ms (TCP, else QUIC); empty when unknown, from 26 Nov 2026
  hello_bytes      DOUBLE    length of the caller's TLS hello, bytes; empty when unknown, from 26 Nov 2026
  counted          INTEGER   1 on an arrival, 0 on an event row; SUM(weight * counted) = requests
  weight           INTEGER   how many requests this row stands for: Analytics Engine stores one row in N when busy

Fields, people/:
  hour             TIMESTAMP start of the hour, UTC
  site             VARCHAR   host asked for
  path             VARCHAR   page or file asked for, no query string
  country          VARCHAR   Cloudflare's country for the reader's address
  requests         INTEGER   requests from browsers the classifier counts as people, weighted

Try it in DuckDB:
  SELECT caller, SUM(weight*counted) n FROM 'https://motdang.net/voight-kampff/data/bots/2026-11-25.parquet'
  GROUP BY caller ORDER BY n DESC LIMIT 20;
Or in the page itself: https://motdang.net/voight-kampff/#ask

Licence: CC BY 4.0. Credit: motdang.net Voight-Kampff, https://motdang.net/voight-kampff/
