上传数据
============================
注意，所有的时间(key,start,end)用标准时间格式，例如：2012-03-15T16:13:14 或者
2012-03-15 16:13:14。
开关传感器
--------------------------------------------------
| url   | 作用   |  方法  |  请求参数 | 返回参数  |
| -------- | -----:  | :----:  | :----:  |:----:  |
| /API/sensor/{sensor_id}/datapoint/|修改|GET|value| json = {'msg'} |
| /API/sensor/{sensor_id}/datapoint/get/|查询|GET||json = {'msg', 'value'}|

数值传感器
--------------------------------------------------
| url   | 作用   |  方法  |  请求参数 | 返回参数  |
| -------- | :-----:  | :----:  | :----:  |:----:  |
| /API/sensor/{sensor_id}/datapoint/|新增|GET，POST|GET（key，value）or POST（body : {'key','value'} or [{'key','value'}] | json = {'msg'} |
| /API/sensor/{sensor_id}/datapoint/get/|查询|GET|key|json = {'msg', 'value}|
| /API/sensor/{sensor_id}/datapoint/edit/|修改|GET|key，value| json = {'msg'} |
| /API/sensor/{sensor_id}/datapoint/remove/|删除|GET|key| json = {'msg'} |
| /API/sensor/{sensor_id}/datapoint/history/|历史（时间段）|GET|start,end,interval(间隔，单位秒）| json = {'msg'， 'datapoint'=[{'value','key'}] |
| /API/sensor/{sensor_id}/datapoint/history/|历史（返回最新前二十条）|GET|| json = {'msg'， 'datapoint'=[{'value','key'}]} |
json data could be like this：
```
    {
      "key":"2012-03-15T16:13:14",
      "value":294.34
    }
```
or
```
    [
      {"key": "2012-06-15T14:00:00", "value":315.01},
      {"key": "2012-06-15T14:00:10", "value":316.23},
      {"key": "2012-06-15T14:00:20", "value":317.26},
      {"key": "2012-06-15T14:00:30", "value":318},
      {"key": "2012-06-15T14:00:40", "value":317}
    ]
```

图像传感器
--------------------------------------------------
暂未测试

Example
--------------------------------------------------
在sh/datapoint_tools下有数据的测试脚本，可以用来增删数据，也可以用来做例子。
[开关传感的读写脚本](https://github.com/wzyy2/PiBox/tree/master/PiBox/sh/datapoint_tools/switch.py)
```
    python switch.py
```
[数据传感的CURD脚本](https://github.com/wzyy2/PiBox/tree/master/PiBox/sh/datapoint_tools/num.py)
```
    python num.py
```

只要对这些脚本稍加修改，另外加上一些python代码就可以通过以下思路，实现自己的本地智能家居：
* 通过串口等通讯模块，关联单片机，然后关联上native yeelink。
* 通过驱动读取树莓派所接模块的数据，然后上传数据到native yeelink。

CallBack
============================
Explain
--------------------------------------------------
Callback file是传感器满足条件后被调用的python代码文件名（比如callback_file=gpio.py)，同时文件需要放在PiBox/sh/callback/下。
不使用的话只需提供不存在的文件名或者保持为空即可。

对开关传感器来说，每一次开关数值点发生修改，callback_file都会被按如下格式调用

        python callback_file sensor.name swtich_status(1 or 0)
        
对数值传感器来说,每一次新增或者修改的数值都会检查条件，如果满足条件，callback_file都会被按如下格式调用

        python callback_file sensor.name value（溢出的数值） key（时间）
        
Example
--------------------------------------------------
在sh/callback下有两个文件实例可以直接使用
### gpio.py

    import os,sys
    
    cwd = os.path.dirname(os.path.abspath(__file__)) + '/..' + '/..'
    sys.path.append(os.path.join(cwd, 'PiHome'))
    
    from common import linux_gpio
    
    print 'Sensor:', str(sys.argv[1])
    print 'Status:', str(sys.argv[2])
    
    GPIO_NUM = 23
    
    print 'GPIO_BCM_NUM:', GPIO_NUM
    
    gpio = linux_gpio.gpio(GPIO_NUM)
    gpio.gpio_export()
    gpio.write_gpio_direction('out')
    gpio.write_gpio_value(sys.argv[2])
在需要的开关传感器下填入gpio.py,将文件的GPIO_NUM改成所需的gpio编号，每次开关状态改变，就会带动gpio改变。
直接使用这个脚本就可以通过web来控制简单电器

### callout.py

    import os,sys

    print 'Sensor:', str(sys.argv[1])
    print 'Value:', str(sys.argv[2])
    print 'key:', str(sys.argv[3])
在需要的数值传感器下填入callout.py和条件,数值发生改变时条件若满足，会发送debug信息到log里。
如果在下面按网上教程加入发邮件或者发微博代码，就可以实现监控报警的效果。