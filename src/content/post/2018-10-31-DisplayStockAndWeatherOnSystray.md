+++
title = "DisplayStockAndWeatherOnSystray"
date = "2018-10-31T08:30:55+08:00"
description = "DisplayStockAndWeatherOnSystray"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Weather
We need lain for displaying weather widget on systray, so first we clone the
lain's source code into our awesome configuration directory:    

```
$ git clone https://github.com/lcpz/lain.git ~/.config/awesome/lain
```
I refered the `awesome-copycats` theme for displaying the weather.    

```
$ git clone --recursive https://github.com/lcpz/awesome-copycats.git
```
Refers to `blackburn` theme:    

```
+ local lain  = require("lain")
+ 
+ -- Weather
+ theme.weather = lain.widget.weather({
+     city_id = 2643743, -- placeholder (London)
+     settings = function()
+         units = math.floor(weather_now["main"]["temp"])
+         widget:set_markup(" " .. units .. " ")
+     end
+ })
```
Read the lain's weather widget implementation:    

```
$ vim lain/widget/weather.lua
 local current_call          = args.current_call  or "curl -s 'http://api.openweathermap.org/data/2.5/weather?id=%s&units=%s&lang=%s&APPID=%s'"
 local forecast_call         = args.forecast_call or "curl -s 'http://api.openweathermap.org/data/2.5/forecast/daily?id=%s&units=%s&lang=%s&cnt=%s&APPID=%s'"
```
We know the weather api is from openweathermap.org. We search `Guangzhou`:    

![/images/2018_10_31_08_57_10_756x308.jpg](/images/2018_10_31_08_57_10_756x308.jpg)

Result:   

![/images/2018_10_31_08_57_25_734x250.jpg](/images/2018_10_31_08_57_25_734x250.jpg)

Detailed weather information for Guangzhou:    

![/images/2018_10_31_08_57_48_1114x629.jpg](/images/2018_10_31_08_57_48_1114x629.jpg)

From the URL we know the id is 1809858, so we replace the `2643743` by
`1809858`, restart the awesome, and we got the weather widget displayed on
Awesome's Systray.    

![/images/2018_10_31_08_58_54_285x75.jpg](/images/2018_10_31_08_58_54_285x75.jpg)

### Stock
I refered following url for writing the script and display it on awesome
systray:    

http://leemeng0x61.github.io/blog/2011/03/25/awesome-%E8%84%9A%E6%9C%AC%E4%B8%AD%E5%8A%A0%E5%85%A5%E5%A4%A9%E6%B0%94/

via following command you could fetch the Shanghai Stock market value:    

```
# curl http://hq.sinajs.cn/list=s_sh000001
var hq_str_s_sh000001="��ָ֤��,2567.9780,-0.0701,-0.00,0,0";
```
So our function for fetching the data is:    

```
$ vim ~/.config/awesome/rc.lua
+ ------------------------fetchShanghaiStock-------------------------
+ -- fetchShanghaiStock(), function for fetching the A Stock of china, Shanghai.
+ -- Return value is substr of 'var hq_str_s_sh000001="上证指数,2568.0481,25.9448,1.02,1666823,15238593";'
+ -- words[2]: Current Value
+ -- words[3]: +/- Value
+ -- words[4]: +/- Value in percentage
+ function fetchShanghaiStock()
+     local url = "http://hq.sinajs.cn/list=s_sh000001"
+     local con, ret = http.request(url)
+     if con == nil then
+         print ("nil")
+         return nil
+     else
+         local words = {}
+         for word in con:gmatch('[^,]+') do
+             table.insert(words,word)
+         end
+         -- print (con)
+         -- print (words[2])
+         -- print (words[3])
+         -- print (words[4])
+         return words
+     end
+ end
```
We use `,` for separating the returned string, and we care the 2nd/3rd/4th
number, send them into an array, and return back.    

Now in awesome's configuration file we use vicious for displaying our fetched
number:    

```
$ vim ~/.config/awesome/rc.lua
+ -- for fetching the China Shanghai Stock
+ local http = require("socket.http")
+ local vicious = require("vicious")

+ stockwidget = wibox.widget.textbox()
+     vicious.register(stockwidget, vicious.widgets.uptime,
+       function (widget, args)
+ 	local l = fetchShanghaiStock()
+ 	return '<span color="brown">A股:</span><span  color="orange">'..l[2]..'|'..l[3]..'|'..l[4]..'</span>'
+       end, 610)


        s.mytasklist, -- Middle widget
        { -- Right widgets
            layout = wibox.layout.fixed.horizontal,
            mykeyboardlayout,
            wibox.widget.systray(),
	    myweather.icon,
            myweather.widget,
+	    stockwidget,
            mytextclock,
            s.mylayoutbox,
        },
    }
```
We use an uptime widget for displaying the fetched content(Calling the
fetchShanghaiStock()), now the result will be displayed like following on
awesome systray:    

![/images/2018_10_31_09_21_36_485x106.jpg](/images/2018_10_31_09_21_36_485x106.jpg)

Generally when the value rise, displaying color will be red, or be green, so
we add following logic for our return value:    

```
	local l = fetchShanghaiStock()
+	if string.match(l[4], "-") then
+	  return '<span color="brown">A股:</span><span  color="green">'..l[2]..'|'..l[3]..'|'..l[4]..'</span>'
+	else
+	  return '<span color="brown">A股:</span><span  color="red">'..l[2]..'|'..l[3]..'|'..l[4]..'</span>'
+	end
      end, 610)
```

Rise:    

![/images/2018_10_31_09_30_23_291x67.jpg](/images/2018_10_31_09_30_23_291x67.jpg)

Lower(Picture to be captured):    

Now we could easily view weather and stock on our awesome systray, but, after
all, it's only systray, how could we display more data on desktop widget, just
link conky did? So next step I will investigate the awesome desktop widgets.   

Todo: also you could adjust the A stock price for BitCoin, or other values.  

Bitcoin price api:    

```
{
  "USD" : {"15m" : 6302.44, "last" : 6302.44, "buy" : 6302.44, "sell" : 6302.44, "symbol" : "$"},
  "AUD" : {"15m" : 8876.47, "last" : 8876.47, "buy" : 8876.47, "sell" : 8876.47, "symbol" : "$"},
  "BRL" : {"15m" : 23295.01, "last" : 23295.01, "buy" : 23295.01, "sell" : 23295.01, "symbol" : "R$"},
  "CAD" : {"15m" : 8268.87, "last" : 8268.87, "buy" : 8268.87, "sell" : 8268.87, "symbol" : "$"},
  "CHF" : {"15m" : 6336.09, "last" : 6336.09, "buy" : 6336.09, "sell" : 6336.09, "symbol" : "CHF"},
  "CLP" : {"15m" : 4370777.09, "last" : 4370777.09, "buy" : 4370777.09, "sell" : 4370777.09, "symbol" : "$"},
  "CNY" : {"15m" : 43910.64, "last" : 43910.64, "buy" : 43910.64, "sell" : 43910.64, "symbol" : "¥"},
  "DKK" : {"15m" : 41449.62, "last" : 41449.62, "buy" : 41449.62, "sell" : 41449.62, "symbol" : "kr"},
  "EUR" : {"15m" : 5543.2, "last" : 5543.2, "buy" : 5543.2, "sell" : 5543.2, "symbol" : "€"},
  "GBP" : {"15m" : 4959.24, "last" : 4959.24, "buy" : 4959.24, "sell" : 4959.24, "symbol" : "£"},
  "HKD" : {"15m" : 49442.34, "last" : 49442.34, "buy" : 49442.34, "sell" : 49442.34, "symbol" : "$"},
  "INR" : {"15m" : 464237.87, "last" : 464237.87, "buy" : 464237.87, "sell" : 464237.87, "symbol" : "₹"},
  "ISK" : {"15m" : 764925.92, "last" : 764925.92, "buy" : 764925.92, "sell" : 764925.92, "symbol" : "kr"},
  "JPY" : {"15m" : 708620.86, "last" : 708620.86, "buy" : 708620.86, "sell" : 708620.86, "symbol" : "¥"},
  "KRW" : {"15m" : 7186217.64, "last" : 7186217.64, "buy" : 7186217.64, "sell" : 7186217.64, "symbol" : "₩"},
  "NZD" : {"15m" : 9599.02, "last" : 9599.02, "buy" : 9599.02, "sell" : 9599.02, "symbol" : "$"},
  "PLN" : {"15m" : 24064.93, "last" : 24064.93, "buy" : 24064.93, "sell" : 24064.93, "symbol" : "zł"},
  "RUB" : {"15m" : 412854.07, "last" : 412854.07, "buy" : 412854.07, "sell" : 412854.07, "symbol" : "RUB"},
  "SEK" : {"15m" : 57818.99, "last" : 57818.99, "buy" : 57818.99, "sell" : 57818.99, "symbol" : "kr"},
  "SGD" : {"15m" : 8732.86, "last" : 8732.86, "buy" : 8732.86, "sell" : 8732.86, "symbol" : "$"},
  "THB" : {"15m" : 209776.78, "last" : 209776.78, "buy" : 209776.78, "sell" : 209776.78, "symbol" : "฿"},
  "TWD" : {"15m" : 195139.36, "last" : 195139.36, "buy" : 195139.36, "sell" : 195139.36, "symbol" : "NT$"}
}%
```
Or CNY to USD?   
