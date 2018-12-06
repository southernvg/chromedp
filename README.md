## Installing

Install in the usual Go way:

```sh
go get -u github.com/chromedp/chromedp
```

## Callback_Function For Events

There is no way for me to add callback_function for the events from chrome devtools,so i just add it and delete something  i think useless(Raw js should exec in the Page context),fixed something,if u have good question pls tell me !


## Examples

CallbackFunc(method string,func(interface{},\*TargetHandler))

u can add a event callbackfunc like that way

```go
package main

import(
	cdp  "github.com/chromedp/chromedp"
	"github.com/chromedp/cdproto/network"
	"github.com/chromedp/chromedp/runner"
	"log"
	"time"
	"github.com/fatih/color"
	"context"

)


func main() {

	var err error

	// create context
	ctxt, cancel:= context.WithCancel(context.Background())

	defer cancel()

	// create chrome instance cdp.WithLog(log.Printf)
	c, err := cdp.New(ctxt,cdp.WithRunnerOptions(
    runner.Flag("headless", true),
    runner.Flag("disable-gpu", true),
    runner.Flag("no-first-run", true),
    runner.Flag("no-default-browser-check", true),
    runner.Flag("no-sandbox",true),
    runner.RemoteDebuggingPort(9222),
))
	if err != nil {
		log.Fatal(err)
	}

	err = c.Run(ctxt, task(ctxt))
	if err != nil {
		log.Fatal(err)
	}

	err = c.Shutdown(ctxt)

	if err != nil {
		log.Fatal(err)
	}

	err = c.Wait()
	
	if err != nil {

		log.Println(err)

	}



}

func task(ctx context.Context)cdp.Tasks{
	return cdp.Tasks{
		cdp.CallbackFunc("Network.requestWillBeSent", func(param interface{},handler *cdp.TargetHandler){
					data := param.(*network.EventRequestWillBeSent)
					color.Red(data.Request.URL)				
					}),
		cdp.Navigate(`https://www.baidu.com`),
		cdp.Sleep(10* time.Second),
	}
}
```

the handler \*cdp.Targethandler is for callbackfunc does next command

```go
package main

import(
	cdp  "github.com/chromedp/chromedp"
	"github.com/chromedp/cdproto/network"
	"github.com/chromedp/chromedp/runner"
	"log"
	"time"
	"github.com/fatih/color"
	"context"

)


func main() {

	var err error

	// create context
	ctxt, cancel:= context.WithCancel(context.Background())

	defer cancel()

	// create chrome instance cdp.WithLog(log.Printf)
	c, err := cdp.New(ctxt,cdp.WithRunnerOptions(
    runner.Flag("headless", true),
    runner.Flag("disable-gpu", true),
    runner.Flag("no-first-run", true),
    runner.Flag("no-default-browser-check", true),
    runner.Flag("no-sandbox",true),
    runner.RemoteDebuggingPort(9222),
))
	if err != nil {
		log.Fatal(err)
	}

	err = c.Run(ctxt, task(ctxt))
	if err != nil {
		log.Fatal(err)
	}

	err = c.Shutdown(ctxt)

	if err != nil {
		log.Fatal(err)
	}

	err = c.Wait()
	
	if err != nil {

		log.Println(err)

	}



}

func task(ctx context.Context)cdp.Tasks{
	return cdp.Tasks{
		cdp.CallbackFunc("Network.requestWillBeSent", func(param interface{},handler *cdp.TargetHandler){
					data := param.(*network.EventRequestWillBeSent)
					color.Red(data.Request.URL)				
					}),
		network.SetRequestInterception([]*network.RequestPattern{
					&network.RequestPattern{
					URLPattern:"*",ResourceType:"Document",
					},
				}),
		cdp.CallbackFunc("Network.requestIntercepted", func(param interface{},handler *cdp.TargetHandler){
						data := param.(*network.EventRequestIntercepted)
				
						json_byte,_:= data.MarshalJSON()

						var out bytes.Buffer

						 _ = json.Indent(&out, json_byte,"","\t")

						fmt.Println(out.String())

						network.ContinueInterceptedRequest(data.InterceptionID).WithHeaders(network.Headers{"key":"val",}).Do(ctx,handler)  //add hander for each dom request

					}),
		cdp.Navigate(`https://www.baidu.com`),
		cdp.Sleep(10* time.Second),
	}
}
``` 


