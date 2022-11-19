### satang-go

A Golang SDK for [satang](https://satangcorp.com/) API.

[![Build Status](https://app.travis-ci.com/tanakorn0314/satang-go.svg?branch=main)](https://travis-ci.org/tanakorn0314/satang-go)
<!-- [![GoDoc](https://godoc.org/github.com/tanakorn0314/satang-go?status.svg)](https://godoc.org/github.com/tanakorn0314/satang-go)
[![Go Report Card](https://goreportcard.com/badge/github.com/tanakorn0314/satang-go)](https://goreportcard.com/report/github.com/tanakorn0314/satang-go)
[![codecov](https://codecov.io/gh/tanakorn0314/satang-go/branch/master/graph/badge.svg)](https://codecov.io/gh/tanakorn0314/satang-go) -->

All the REST APIs listed in [satang API document](https://docs.satangcorp.com/) are implemented, as well as the websocket APIs.

For best compatibility, please use Go >= 1.9.

Make sure you have read satang API document before continuing.

### API List

Name | Description | Status
------------ | ------------ | ------------
[web-socket-streams.md](https://docs.satangcorp.com/#e0164e24-6925-46f4-be6e-bd43c81ae2cb) | Details on available streams and payloads | <input type="checkbox" checked>  Implemented

### Installation

```shell
go get github.com/tanakorn0314/satang-go/v2
```

### Importing

```golang
import (
    "github.com/tanakorn0314/satang-go/v2"
)
```

### Documentation

[![GoDoc](https://godoc.org/github.com/tanakorn0314/satang-go?status.svg)](https://godoc.org/github.com/tanakorn0314/satang-go)

### REST API

#### Setup

Init client for API services. Get APIKey/SecretKey from your satang account.

```golang
var (
    apiKey = "your api key"
    secretKey = "your secret key"
)
client := satang.NewClient(apiKey, secretKey)
```

A service instance stands for a REST API endpoint and is initialized by client.NewXXXService function.

Simply call API in chain style. Call Do() in the end to send HTTP request.

Following are some simple examples, please refer to [godoc](https://godoc.org/github.com/tanakorn0314/satang-go) for full references.

#### Create Order

```golang
order, err := client.NewCreateOrderService().Symbol("usdt_thb").
        Side(satang.SideTypeBuy).Type(satang.OrderTypeLimit).
        TimeInForce(satang.TimeInForceTypeGTC).Quantity("5").
        Price("0.0030000").Do(context.Background())
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println(order)

// Use Test() instead of Do() for testing.
```

#### Get Order

```golang
order, err := client.NewGetOrderService().Symbol("usdt_thb").
    OrderID(4432844).Do(context.Background())
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println(order)
```

#### Cancel Order

```golang
_, err := client.NewCancelOrderService().Symbol("usdt_thb").
    OrderID(4432844).Do(context.Background())
if err != nil {
    fmt.Println(err)
    return
}
```

#### List Open Orders

```golang
openOrders, err := client.NewListOpenOrdersService().Symbol("usdt_thb").
    Do(context.Background())
if err != nil {
    fmt.Println(err)
    return
}
for _, o := range openOrders {
    fmt.Println(o)
}
```

#### List Orders

```golang
orders, err := client.NewListOrdersService().Symbol("usdt_thb").
    Do(context.Background())
if err != nil {
    fmt.Println(err)
    return
}
for _, o := range orders {
    fmt.Println(o)
}
```

#### List Ticker Prices

```golang
prices, err := client.NewListPricesService().Do(context.Background())
if err != nil {
    fmt.Println(err)
    return
}
for _, p := range prices {
    fmt.Println(p)
}
```

#### Show Depth

```golang
res, err := client.NewDepthService().Symbol("usdt_thb").
    Do(context.Background())
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println(res)
```

#### List Klines

```golang
klines, err := client.NewKlinesService().Symbol("usdt_thb").
    Interval("15m").Do(context.Background())
if err != nil {
    fmt.Println(err)
    return
}
for _, k := range klines {
    fmt.Println(k)
}
```

#### List Aggregate Trades

```golang
trades, err := client.NewAggTradesService().
    Symbol("usdt_thb").StartTime(1508673256594).EndTime(1508673256595).
    Do(context.Background())
if err != nil {
    fmt.Println(err)
    return
}
for _, t := range trades {
    fmt.Println(t)
}
```

#### Get Account

```golang
res, err := client.NewGetAccountService().Do(context.Background())
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println(res)
```

#### Start User Stream

```golang
res, err := client.NewStartUserStreamService().Do(context.Background())
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println(res)
```

### Websocket

You don't need Client in websocket API. Just call satang.WsXxxServe(args, handler, errHandler).

> For delivery API you can use `delivery.WsXxxServe(args, handler, errHandler)`.

#### Depth

```golang
wsDepthHandler := func(event *satang.WsDepthEvent) {
    fmt.Println(event)
}
errHandler := func(err error) {
    fmt.Println(err)
}
doneC, stopC, err := satang.WsDepthServe("usdt_thb", wsDepthHandler, errHandler)
if err != nil {
    fmt.Println(err)
    return
}
// use stopC to exit
go func() {
    time.Sleep(5 * time.Second)
    stopC <- struct{}{}
}()
// remove this if you do not want to be blocked here
<-doneC
```

#### Kline

```golang
wsKlineHandler := func(event *satang.WsKlineEvent) {
    fmt.Println(event)
}
errHandler := func(err error) {
    fmt.Println(err)
}
doneC, _, err := satang.WsKlineServe("usdt_thb", "1m", wsKlineHandler, errHandler)
if err != nil {
    fmt.Println(err)
    return
}
<-doneC
```

#### Aggregate

```golang
wsAggTradeHandler := func(event *satang.WsAggTradeEvent) {
    fmt.Println(event)
}
errHandler := func(err error) {
    fmt.Println(err)
}
doneC, _, err := satang.WsAggTradeServe("usdt_thb", wsAggTradeHandler, errHandler)
if err != nil {
    fmt.Println(err)
    return
}
<-doneC
```

#### User Data

```golang
wsHandler := func(message []byte) {
    fmt.Println(string(message))
}
errHandler := func(err error) {
    fmt.Println(err)
}
doneC, _, err := satang.WsUserDataServe(listenKey, wsHandler, errHandler)
if err != nil {
    fmt.Println(err)
    return
}
<-doneC
```

#### Setting Server Time

Your system time may be incorrect and you may use following function to set the time offset based off Satang Server Time:

```golang
// use the client future for Futures
client.NewSetServerTimeService().Do(context.Background())
```

Or you can also overwrite the `TimeOffset` yourself:

```golang
client.TimeOffset = 123
```