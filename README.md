[![GoDoc](https://godoc.org/github.com/thinkgos/gomodbus?status.svg)](https://godoc.org/github.com/thinkgos/gomodbus)
[![Build Status](https://www.travis-ci.org/thinkgos/gomodbus.svg?branch=master)](https://www.travis-ci.org/thinkgos/gomodbus)
[![codecov](https://codecov.io/gh/thinkgos/gomodbus/branch/master/graph/badge.svg)](https://codecov.io/gh/thinkgos/gomodbus)
![Action Status](https://github.com/thinkgos/gomodbus/workflows/Go/badge.svg)
[![Go Report Card](https://goreportcard.com/badge/github.com/thinkgos/gomodbus)](https://goreportcard.com/report/github.com/thinkgos/gomodbus)
[![Licence](https://img.shields.io/github/license/thinkgos/gomodbus)](https://raw.githubusercontent.com/thinkgos/gomodbus/master/LICENSE)


### go modbus Supported formats

- modbus TCP Client
- modbus Serial(RTU,ASCII) Client
- modbus TCP Server

### 特性

- 临时对象缓冲池,减少内存分配
- 快速编码,解码
- interface设计,提供扩展性
- 简单的丰富的API

大量参考了!为了用于生产环境[goburrow](https://github.com/goburrow/modbus)

### Supported functions

---

Bit access:
*   Read Discrete Inputs
*   Read Coils
*   Write Single Coil
*   Write Multiple Coils

16-bit access:
*   Read Input Registers
*   Read Holding Registers
*   Write Single Register
*   Write Multiple Registers
*   Read/Write Multiple Registers
*   Mask Write Register
*   Read FIFO Queue

### Example

---

```go
	p := modbus.NewTCPClientProvider("192.168.199.188:502")
	client := modbus.NewClient(p)
	client.LogMode(true)
	err := client.Connect()
	if err != nil {
		fmt.Println("connect failed, ", err)
		return
	}

	defer client.Close()
    fmt.Println("starting")
	for {
		results, err := client.ReadCoils(1, 0, 10)
		if err != nil {
			fmt.Println(err.Error())
		} else {
			fmt.Printf("ReadCoils % x", results)
		}
		time.Sleep(time.Second * 5)
	}
```

```go
    // modbus RTU/ASCII Client
    p := modbus.NewRTUClientProvider("")
    p.Address = "COM5"
    p.BaudRate = 115200
	p.DataBits = 8
	p.Parity = "N"
	p.StopBits = 1
	client := modbus.NewClient(p)
	client.LogMode(true)
	err := client.Connect()
	if err != nil {
		fmt.Println("connect failed, ", err)
		return
	}

	defer client.Close()
    fmt.Println("starting")
	for {
		results, err := client.ReadCoils(1, 0, 10)
		if err != nil {
			fmt.Println(err.Error())
		} else {
			fmt.Printf("ReadDiscreteInputs %#v\r\n", results)
		}
		time.Sleep(time.Second * 5)
	}
```

```go
    // modbus TCP Server
	srv := modbus.NewTCPServer(":502")
	srv.Logger = log.New(os.Stdout, "modbus", log.Ltime)
	srv.LogMode(true)

	srv.AddNode(modbus.NewNodeRegister(
		1,
		0, 10, 0, 10, 
		0, 10, 0, 10,
	))
	err := srv.ListenAndServe(":502")
	if err != nil {
		panic(err)
	}
```

```go
    // mb simple example
    func main() {
        p := modbus.NewRTUClientProvider()
        p.Address = "/dev/ttyUSB0"
        p.BaudRate = 115200
        p.DataBits = 8
        p.Parity = "N"
        p.StopBits = 1
        client := mb.NewClient(p, mb.WitchHandler(&handler{}))
        client.LogMode(true)
        err := client.Start()
        if err != nil {
            panic(err)
        }
    
        err = client.AddGatherJob(mb.Request{
            SlaveID:  1,
            FuncCode: modbus.FuncCodeReadHoldingRegisters,
            Address:  0,
            Quantity: 300,
            ScanRate: time.Second,
        })
    
        if err != nil {
            panic(err)
        }
    
        select {}
    }
    
    type handler struct{}
    
    func (handler) ProcReadCoils(byte, uint16, uint16, []byte)            {}
    func (handler) ProcReadDiscretes(byte, uint16, uint16, []byte)        {}
    func (handler) ProcReadHoldingRegisters(byte, uint16, uint16, []byte) {}
    func (handler) ProcReadInputRegisters(byte, uint16, uint16, []byte)   {}
    func (handler) ProcResult(_ error, result *mb.Result) {
        log.Printf("Tx=%d,Err=%d,SlaveID=%d,FC=%d,Address=%d,Quantity=%d,SR=%dms",
            result.TxCnt, result.ErrCnt, result.SlaveID, result.FuncCode,
            result.Address, result.Quantity, result.ScanRate/time.Millisecond)
    }
```

### References

---

-   [Modbus Specifications and Implementation Guides](http://www.modbus.org/specs.php)
