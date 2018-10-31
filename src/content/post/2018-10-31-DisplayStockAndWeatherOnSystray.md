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
