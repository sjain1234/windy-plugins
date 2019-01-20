windy-plugin-saws-warnings
==========================

This plugin reads geojson data from:  http://www.weathersa.co.za/ and then displays it on Windy.

Plugin uses:  **pickerTools.mjs**,  which exports an object with these methods:

- drag(callbackfx)  //provides picker {lat:lat,lon:lon} coordinates to the cbfx on picker drag.
- fillRightDiv(string||DIV made with document.createElement("div"))  
- fillLeftDiv(string||DIV)
- hideLeftDiv() and showLeftDiv()  //also for RightDiv.  

pickerDiv.less // style the picker left and right divs

