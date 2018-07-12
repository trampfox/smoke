# Smoke

[![Build Status](https://travis-ci.org/bluehoodie/smoke.svg?branch=master)](https://travis-ci.org/bluehoodie/smoke)
[![Go Report Card](https://goreportcard.com/badge/github.com/bluehoodie/smoke)](https://goreportcard.com/report/github.com/bluehoodie/smoke)
[![Coverage Status](https://coveralls.io/repos/github/Tkanos/smoke/badge.svg?branch=master)](https://coveralls.io/github/Tkanos/smoke?branch=master)


A simple application to write and run smoke tests for RESTful APIs.

## Getting Started

### Running

The most convenient way of running this code, especially in a CI environment, is to use the docker image `bluehoodie/smoke`

`docker run -v {YOUR_TESTFILE_LOCATION}:/test bluehoodie/smoke -f /test/{YOUR_TESTFILE} -u http://{YOUR_URL}`

Otherwise, it can be installed locally by running 

`go get github.com/bluehoodie/smoke`

### Usage

``` 
Usage:
  smoke [OPTIONS]

Application Options:
  -v, --verbose  print out full report including successful results
  -f, --file=    file containing the test definition (default: ./smoke_test.json)
  -u, --url=     url endpoint to test (default: http://localhost)
  -p, --port=    port the service is running on
  -t, --timeout= timeout in seconds for each http request made (default: 1)

Help Options:
  -h, --help     Show this help message
```

## Writing a test file

The test file can be either a JSON or YAML map with the following elements:

- `globals`: a map of of keys to values representing variables which can be accessed in all test cases
- `contracts`: a list of user-defined contracts reprenting each test case

The structure of a contract element is a map with the following elements:

- `name`: label for this test case
- `path`: uri endpoint to call for this test. (will be appended to the URL defined in the command)
- `method`: http verb, ie: GET, POST, etc
- `body`: http request body. (optional)
- `headers`: map of header values to add to the http request (optional)
- `locals`: map of variables specific to this test case. will override the global values
- `http_code_is`: integer representing the expected http code in the result
- `response_body_contains`: string representing an expected value within the resulting response body (Can be a regexp begining by r/ example "r/[0-9]*" )

See the `smoke_test.json` and `smoke_test.yaml` files for examples. 

### Variables

Variables can be used in the path, body or header values. The way a variable is called is by wrapping it in `::`, e.g.: `::variable_name::`
 
The order of precedence for looking for variable values is:

1. local variables defined in the contract map ("locals")
2. global variables defined in the outer variables map ("globals")
3. environment variables

If no value is found, then the test will fail.

##### Example

If our test case path is defined this way:

```"path": "/get?foo=::token::&bar=1",```

Here, ```::token::``` will be replaced with whichever value is found. 

## Result

Running a test will result in the following possible exit codes:

- 0 : if the tests run and all tests passed
- 1 : if the tests ran but there were some failed tests.
- 2 : if the tests could not be run (error reading or parsing the json test file)

If any tests failed, some output will be written to stderr with more detail about the failed tests.

If verbose mode is on, a report on all tests will be written to stdout.

## License

MIT. see LICENSE file.