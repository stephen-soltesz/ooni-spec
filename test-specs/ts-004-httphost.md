# Specification version number

2013-02-26-000

# Specification name

HTTP Host test

# Test preconditions

  * An internet connection.
  * That no special treatment is given to the supplied oonib test helper.

For reporting to the backend to work that it is possible for the probe to
establish a connection to the Tor network.

# Expected impact

  * Ability to determine that the transparent HTTP proxy is doing censorship
    based on the HTTP Host header field.

  * Ability to detect the presence of a Transparent HTTP Proxy

  * Ability to detect which logic is being used by the Transparent proxy to
    censor the target sites and if some circumvention strategies are effective.

  * (optional) if the blockpage is specified if the hostname under analysis is
    being blocked.

# Expected inputs

  * A list of hostnames to be tested

  * The IP address (or hostname) of an oonib HTTPReturnJSONHeadersHelper test
    helper running on port 80.

  * (optional) the content of the blockpage to compare against when processing
    the response.

## Semantics

One per line a list of hostnames, for example:

    torproject.org
    ooni.nu

# Test description

For every given hostname we perform the following series of tests. Once every
test is completed we always perform a fixed set of operations to infer the
presence of a transparent HTTP proxy and/or censorship.

We take the response from our request and check to see if it starts with the
character '{', if it does not we consider that a transparent HTTP proxy is
present.

If not we attempt to parse the response as a JSON string, if it does not parse
we consider a that a transparent HTTP proxy is present.

If the JSON string does parse we look for the following dict keys:

  * 'request_headers'

  * 'request_line'

  * 'headers_dict'

If all of them are present we consider that no transparent HTTP proxy is
present.

If a transparent HTTP proxy is present and the user has specified the content
of the censorship blockpage we compare the response with the known blockpage
and check if they match. If they do match then the hostname is maked as
censored.

These operations are done once for every one of the following tests:

## test_send_host_header

We connect to the backend test helper on port 80 and perform a HTTP GET request
with the Host header field set to the target hostname.

## test_filtering_via_fuzzy_matching

The Host header field contains the hostname prefixed by 10 random characters
and postfixed by 10 random characters.

The purpose of this is to determine if censorship is being triggerred by fuzzy
matching.

## test_filtering_of_subdomain

The Host header field contains a random 10 character subdomain of the target
hostname (`ninechars1.example.com`).

The purpose of this is to determine if also subdomains are being censored.


## test_filtering_add_tab_to_host

The Host header field contains the subdomain postfixed by the tab character
`\t`.

The purpose of this is to determine if by appending a tab character the filter
is being bypassed.

## test_filtering_prepend_newline_to_method

The HTTP Request Line is prefixed with a newline character `\n`.

The purpose of this is to determine if this is a valid filter bypassing
strategy.

XXX move this to a separate test as it does not have much to do with the HTTP
Host field.

# Expected output

## Parent data format

df-001-httpt

## Semantics

'censored': true|false|null

  If the site being analyzed appears to be censored.

  If the content of the blockpage is specified we make an evaluation of
  censorship or not based on the response matching it or not.

  If the response contains the expect JSON dict returned from the oonib test
  helper then we consider censorship to not be happening ('censorship': False).

  In all other cases 'censorship' is set to null.

'transparent_http_proxy': true|false

  if we have detected the presence of a transparent HTTP proxy or not.

## Possible conclusions

We can say that a certain site is blocked or not and looking at the result we
can understand which censorship bypassing strategies have worked and therefore
understand which censorship device the one being analyzed may be.
## Example output sample

      ###########################################
      # OONI Probe Report for HTTP Host test
      # Tue Feb 26 16:27:07 2013
      ###########################################
      ---
      options:
        collector: httpo://nkvphnp3p6agi5qq.onion
        help: 0
        logfile: null
        no-default-reporter: 0
        parallelism: '10'
        pcapfile: null
        reportfile: null
        resume: 0
        subargs: [-b, 'http://93.95.227.200:80', -f, example_inputs/http_host_file.txt]
        test: nettests/manipulation/http_host.py
      probe_asn: null
      probe_cc: null
      probe_ip: null
      software_name: ooniprobe
      software_version: 0.0.10
      start_time: 1361888827.0
      test_name: HTTP Host
      test_version: 0.2.3
      ...
      ---
      input: torproject.org
      agent: agent
      censored: false
      requests:
      - request:
          body: null
          headers:
          - - Host
            - [oiZkiyEm4w.torproject.org]
          method: GET
          url: http://93.95.227.200:80
        response:
          body: '{"headers_dict": {"Connection": ["close"], "Host": ["oiZkiyEm4w.torproject.org"]},
            "request_line": "GET / HTTP/1.1", "request_headers": [["Connection", "close"],
            ["Host", "oiZkiyEm4w.torproject.org"]]}'
          code: 200
          headers: []
      socksproxy: null
      trans_http_proxy: false
    test_name: test_filtering_of_subdomain
    test_runtime: 0.27028393745422363
    test_started: 1361892427.285084
    ...
    ---
    input: torproject.org
    agent: agent
    censored: false
    requests:
    - request:
        body: null
        headers:
        - - Host
          - [torproject.org]
        method: '

          GET'
        url: http://93.95.227.200:80
      response:
        body: '{"headers_dict": {"Connection": ["close"], "Host": ["torproject.org"]},
          "request_line": "\nGET / HTTP/1.1", "request_headers": [["Connection", "close"],
          ["Host", "torproject.org"]]}'
        code: 200
        headers: []
    socksproxy: null
    trans_http_proxy: false
    test_name: test_filtering_prepend_newline_to_method
    test_runtime: 0.27550721168518066
    test_started: 1361892427.287634
    ...
    ---
    input: ooni.nu
    agent: agent
    censored: false
    requests:
    - request:
        body: null
        headers:
        - - Host
          - [zzEtRtreZiooni.nuEBQm4yXrpR]
        method: GET
        url: http://93.95.227.200:80
      response:
        body: '{"headers_dict": {"Connection": ["close"], "Host": ["zzEtRtreZiooni.nuEBQm4yXrpR"]},
          "request_line": "GET / HTTP/1.1", "request_headers": [["Connection", "close"],
          ["Host", "zzEtRtreZiooni.nuEBQm4yXrpR"]]}'
        code: 200
        headers: []
      socksproxy: null
      trans_http_proxy: false
      test_name: test_filtering_via_fuzzy_matching
      test_runtime: 0.9233081340789795
      test_started: 1361892427.290093
      ...
      ---
      input: torproject.org
      agent: agent
      censored: false
      requests:
      - request:
          body: null
          headers:
          - - Host
            - [torproject.org]
          method: GET
          url: http://93.95.227.200:80
        response:
          body: '{"headers_dict": {"Connection": ["close"], "Host": ["torproject.org"]},
            "request_line": "GET / HTTP/1.1", "request_headers": [["Connection", "close"],
            ["Host", "torproject.org"]]}'
          code: 200
          headers: []
      socksproxy: null
      trans_http_proxy: false
      test_name: test_send_host_header
      test_runtime: 0.9341740608215332
      test_started: 1361892427.285841
      ...
      ---
      input: ooni.nu
      agent: agent
      censored: false
      requests:
      - request:
          body: null
          headers:
          - - Host
            - ["ooni.nu\t"]
          method: GET
          url: http://93.95.227.200:80
        response:
          body: '{"headers_dict": {"Connection": ["close"], "Host": ["ooni.nu"]}, "request_line":
            "GET / HTTP/1.1", "request_headers": [["Connection", "close"], ["Host", "ooni.nu"]]}'
          code: 200
          headers: []
      socksproxy: null
      trans_http_proxy: false
      test_name: test_filtering_add_tab_to_host
      test_runtime: 1.353187084197998
      test_started: 1361892427.289563
      ...
      ---
      input: ooni.nu
      agent: agent
      censored: false
      requests:
      - request:
          body: null
          headers:
          - - Host
            - [ooni.nu]
          method: '

            GET'
          url: http://93.95.227.200:80
        response:
          body: '{"headers_dict": {"Connection": ["close"], "Host": ["ooni.nu"]}, "request_line":
            "\nGET / HTTP/1.1", "request_headers": [["Connection", "close"], ["Host",
            "ooni.nu"]]}'
          code: 200
          headers: []
      socksproxy: null
      trans_http_proxy: false
      test_name: test_filtering_prepend_newline_to_method
      test_runtime: 1.3599870204925537
      test_started: 1361892427.290647
      ...
      ---
      input: ooni.nu
      agent: agent
      censored: false
      requests:
      - request:
          body: null
          headers:
          - - Host
            - [ooni.nu]
          method: GET
          url: http://93.95.227.200:80
        response:
          body: '{"headers_dict": {"Connection": ["close"], "Host": ["ooni.nu"]}, "request_line":
            "GET / HTTP/1.1", "request_headers": [["Connection", "close"], ["Host", "ooni.nu"]]}'
          code: 200
          headers: []
      socksproxy: null
      trans_http_proxy: false
      test_name: test_send_host_header
      test_runtime: 2.463576078414917
      test_started: 1361892427.289032
      ...
      ---
      input: ooni.nu
      agent: agent
      censored: false
      requests:
      - request:
          body: null
          headers:
          - - Host
            - [rnsLc4tA6s.ooni.nu]
          method: GET
          url: http://93.95.227.200:80
        response:
          body: '{"headers_dict": {"Connection": ["close"], "Host": ["rnsLc4tA6s.ooni.nu"]},
            "request_line": "GET / HTTP/1.1", "request_headers": [["Connection", "close"],
            ["Host", "rnsLc4tA6s.ooni.nu"]]}'
          code: 200
          headers: []
      socksproxy: null
      trans_http_proxy: false
      test_name: test_filtering_of_subdomain
      test_runtime: 3.4960269927978516
      test_started: 1361892427.288356
      ...
      ---
      input: torproject.org
      agent: agent
      censored: false
      requests:
      - request:
          body: null
          headers:
          - - Host
            - [NNzbWQGgNltorproject.org5uosHwtVjn]
          method: GET
          url: http://93.95.227.200:80
        response:
          body: '{"headers_dict": {"Connection": ["close"], "Host": ["NNzbWQGgNltorproject.org5uosHwtVjn"]},
            "request_line": "GET / HTTP/1.1", "request_headers": [["Connection", "close"],
            ["Host", "NNzbWQGgNltorproject.org5uosHwtVjn"]]}'
          code: 200
          headers: []
      socksproxy: null
      trans_http_proxy: false
      test_name: test_filtering_via_fuzzy_matching
      test_runtime: 3.567512035369873
      test_started: 1361892427.286998
      ...
      ---
      input: torproject.org
      agent: agent
      censored: false
      requests:
      - request:
          body: null
          headers:
          - - Host
            - ["torproject.org\t"]
          method: GET
          url: http://93.95.227.200:80
        response:
          body: '{"headers_dict": {"Connection": ["close"], "Host": ["torproject.org"]},
            "request_line": "GET / HTTP/1.1", "request_headers": [["Connection", "close"],
            ["Host", "torproject.org"]]}'
          code: 200
          headers: []
      socksproxy: null
      trans_http_proxy: false
      test_name: test_filtering_add_tab_to_host
      test_runtime: 3.5733180046081543
      test_started: 1361892427.286416
      ...

# Privacy considerations

If the user is behind a transparent HTTP proxy that sets the X-Forwarded-For
header their IP address will end up being part of the final report.

