# go-mockgen

A mock interface code generator.

## Installation

Simply run `go get -u github.com/efritz/go-mockgen/...`.

## Binary Usage

As an example, we generate a mock for the `Retry` interface from the library
[watchdog](https://github.com/efritz/watchdog). If the watchdog library can
be found in the GOPATH, the the following command will generate a file called
`mock_retry.go` with the following content. This assumes that the current
working directory is called *example*.

```bash
$ go-mockgen github.com/efritz/watchdog -i Retry -d .
```

```go
// DO NOT EDIT
// Code generated automatically by github.com/efritz/go-mockgen

package example

import reception "github.com/efritz/reception"

type MockClient struct {
	NewWatcherFunc             func(string) reception.Watcher
	NewWatcherFuncCallCount    int
	NewWatcherFuncCallParams   []ClientNewWatcherParamSet
	RegisterFunc               func(*reception.Service, func(error)) error
	RegisterFuncCallCount      int
	RegisterFuncCallParams     []ClientRegisterParamSet
	ListServicesFunc           func(string) ([]*reception.Service, error)
	ListServicesFuncCallCount  int
	ListServicesFuncCallParams []ClientListServicesParamSet
}

type ClientRegisterParamSet struct {
	Arg0 *reception.Service
	Arg1 func(error)
}

type ClientListServicesParamSet struct {
	Arg0 string
}

type ClientNewWatcherParamSet struct {
	Arg0 string
}

var _ reception.Client = NewMockClient()

func NewMockClient() *MockClient {
	m := &MockClient{}
	m.RegisterFunc = m.defaultRegisterFunc
	m.ListServicesFunc = m.defaultListServicesFunc
	m.NewWatcherFunc = m.defaultNewWatcherFunc
	return m
}

func (m *MockClient) ListServices(v0 string) ([]*reception.Service, error) {
	m.ListServicesFuncCallCount++
	m.ListServicesFuncCallParams = append(m.ListServicesFuncCallParams, ClientListServicesParamSet{v0})
	return m.ListServicesFunc(v0)
}

func (m *MockClient) NewWatcher(v0 string) reception.Watcher {
	m.NewWatcherFuncCallCount++
	m.NewWatcherFuncCallParams = append(m.NewWatcherFuncCallParams, ClientNewWatcherParamSet{v0})
	return m.NewWatcherFunc(v0)
}

func (m *MockClient) Register(v0 *reception.Service, v1 func(error)) error {
	m.RegisterFuncCallCount++
	m.RegisterFuncCallParams = append(m.RegisterFuncCallParams, ClientRegisterParamSet{v0, v1})
	return m.RegisterFunc(v0, v1)
}

func (m *MockClient) defaultRegisterFunc(v0 *reception.Service, v1 func(error)) error {
	return nil
}

func (m *MockClient) defaultListServicesFunc(v0 string) ([]*reception.Service, error) {
	return nil, nil
}

func (m *MockClient) defaultNewWatcherFunc(v0 string) reception.Watcher {
	return nil
}
```

Multiple import paths can be supplied, and a mock will be generated for each
exported interface found in each package (but not subpackages). A whitelist
of packages can be supplied in order to prevent generating unnecessary code
(discussed in the flags section below).

The suggested way to generate mocks for a project is to use go-generate. Then,
if a dependent interface ever changes, running `go generate` on the package will
re-generate the mocks.

```go
package foo

//go:generate go-mockgen -d . -f github.com/efritz/watchdog -i Retry
//go:generate go-mockgen -d . -f github.com/efritz/overcurrent -i Breaker
```

### Flags

The following flags are defined by the binary.

| Name       | Short Flag | Description  |
| ---------- | ---------- | ------------ |
| package    | p          | The name of the generated package. Is the name of target directory if dirname or filename is supplied by default. |
| prefix     |            | A prefix used in the name of each mock struct. Should be TitleCase by convention. |
| interfaces | i          | A whitelist of interfaces to generate given the import paths. |
| filename   | o          | The target output file. All mocks are writen to this file. |
| dirname    | d          | The target output directory. Each mock will be written to a unique file. |
| force      | f          | Do not abort if a write to disk would overwrite an existing file. |
| list       |            | Dry run - print the interfaces found in the given import paths. |

If neither dirname nor filename are supplied, then the generated code is printed to standard out.

## Mock Usage

Each mock can be initialized via the no-argument constructor. This is a valid
implementation of the mocked interface that returns zero values on every function
call. For testing, it may be beneficial to force a return value or side effect when
a particular method of the interface is called. This is supported by re-assigning
the function value in the mock struct to a function defined within your test. This
also allows functions to be monkeypatched in-line, capturing values from the test
method such as communication channels, call counters, and maps in which function
call arguments can be stored.

The following (stripped) example from [reception](https://github.com/efritz/reception)
uses this pattern to mock a connection to Zookeeper, returning an error when attempting
to create an ephemeral znode.

```go
func (s *ZkSuite) TestRegisterError(t sweet.T) {
    conn := NewMockZkConn()
    conn.CreateEphemeralFunc = func(path string, data []byte) error {
        return zk.ErrUnknown
    }

    client := newZkClient(conn)
    err := client.Register(&Service{Name: "s", ID: "1234"}, nil)
    Expect(err).To(Equal(zk.ErrUnknown))
    Expect(conn.CreateEphemeralFuncCallCount).To(Equal(1))
    Expect(conn.CreateEphemeralFuncParams[0].Arg0).To(Equal("s-1234"))
}
```

## License

Copyright (c) 2018 Eric Fritz

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
