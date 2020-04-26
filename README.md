# StreamPublisher
The **StreamPublisher** module for [Wowza Streaming Engine™ media server software](https://www.wowza.com/products/streaming-engine) lets you use a server listener and application module to create a schedule of streams and playlists. Using television as an analogy, a stream is a channel and a playlist is a program with one or more video segments. A schedule can have as many streams (channels) as you want, with as many playlists (programs) as you want, and each playlist can be scheduled to play on a stream at a certain time. If a playlist is scheduled to start on a stream while another playlist is running, the existing playlist is replaced with the new one.

This repo includes a [compiled version](/lib/wse-plugin-streampublisher.jar).

## Prerequisites
Wowza Streaming Engine 4.0.0 or later is required.

## Usage

### Schedules in the Past
If you set a schedule to begin in the past, the playlist plays immediately.
This behavior can be changed by setting the following property to `true`
on either the server or application:
```xml
<Property>
	<Name>streamPublisherPlayPreviousSchedules</Name>
	<Value>false</Value>
	<Type>Boolean</Type>
</Property>
```

### Modules
You can create a schedule with server listener and application module methods, either together or separately, depending on your needs.

You can use the **ServerListenerStreamPublisher** server listener to load a set of scheduled streams on a single application when the media server starts. This procedure keeps the streams running until the server is shut down. If you use this process, the schedule can't be reloaded by using just the server listener.

You can use the **ModuleStreamPublisher** application module on any application to load a set of scheduled streams on that application when the application starts and unload the streams when the application is shut down. The schedule can be reloaded by modifying the SMIL file for the application and then reloading it. The module can provide the reload functionality to the schedule that's configured in the server listener. It can also be used on its own, in separate applications, to provide separate schedules for each application. Each application that runs a schedule must have a **live** stream type.

You can use the **HttpProviderStreamPublisherControl** HttpProvider to load and unload the schedules. To enable the HttpProvider, add the following to your Admin HostPort HttpProvider list.
```
<HTTPProvider>
	<BaseClass>com.wowza.wms.plugin.streampublisher.HttpProviderStreamPublisherControl</BaseClass>
	<RequestFilters>schedules*</RequestFilters>
	<AuthenticationMethod>admin-digest</AuthenticationMethod>
</HTTPProvider>
```
A request to the HttpProvider with no parameters will reload the schedule on the default application defined by the *streamPublisherApplication* server property. The following query parameters can be used.
**appName** - The name of the application to load schedule on.
**appInstName** - The name of the appInstance to load the schedule on.
**action** - *loadSchedule*, *reloadSchedule* or *unloadSchedule*.

If the **action** is *reloadSchedule* or *loadSchedule*, and the appInstance is already running with a schedule then this will be reloaded otherwise the appInstance will be started with a new schedule, **only** if the **action** is *loadSchedule*.
If the **action** is *unloadSchedule*, then the schedule will be unloaded and *loadSchedule* will be required to load it again later.

Load a schedule
```
curl http://localhost:8086/schedules?appName=live&action=loadSchedule
```
Reload a running schedule
```
curl http://localhost:8086/schedules?appName=live&action=reloadSchedule
```
Unload a running schedule
```
curl http://localhost:8086/schedules?appName=live&action=unloadSchedule
```

### Timezones
By default, the schedule will use the local timezone set on the server.
Since many servers use UTC, a different timezone may be specified by either a
custom server/application property or in the smil playlist.

A list of allowable timezone names can be found on the [IANA website](https://www.iana.org/time-zones)
(official list) or [Wikipedia](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).

#### Server Timezone
A default timezone can be set at the server or application level by setting the
following property:
```xml
<Property>
	<Name>streamPublisherTimezoneName</Name>
	<Value>US/Central</Value>
	<Type>String</Type>
</Property>
```
This will be the default for any scheduled items in the server or application.

#### Playlist Timezone
Timezones can also be set by attributes in the `streamschedule.smil` file.
Add an attribute named `timezone` to each `playlist` tag:
```xml
<smil>
<head></head>
<body>
    <stream name="Stream1"></stream>
    <playlist name="pl1" playOnStream="Stream1" repeat="false" timezone="US/Central" scheduled="2020-04-25 16:00:00">
        <video src="mp4:sample.mp4" start="0" length="-1" />
    </playlist>
</body>
</smil>
```

Note that a timezone specified on a playlist will override the [streamPublisherTimezoneName property](#Server-Timezone).

### ModuleLoopUntilLive
To use the included **ModuleLoopUntilLive** application module to loop pre-roll video around a live stream, at least one server-side stream must be configured on the Streaming Engine live application.

## More resources
To use the compiled version of this module, see [Schedule streaming with a Wowza Streaming Engine Java module](https://www.wowza.com/docs/how-to-schedule-streaming-with-wowza-streaming-engine-streampublisher).

For instructions on using the **ModuleLoopUntilLive** application module, see [Loop a pre-roll until a live stream starts with Wowza Streaming Engine Java module](https://www.wowza.com/docs/how-to-loop-a-pre-roll-until-a-live-stream-starts-loopuntillive).

[Wowza Streaming Engine Server-Side API Reference](https://www.wowza.com/resources/serverapi/)

[How to extend Wowza Streaming Engine using the Wowza IDE](https://www.wowza.com/docs/how-to-extend-wowza-streaming-engine-using-the-wowza-ide)

Wowza Media Systems™ provides developers with a platform to create streaming applications and solutions. See [Wowza Developer Tools](https://www.wowza.com/developer) to learn more about our APIs and SDK.

## Contact
[Wowza Media Systems, LLC](https://www.wowza.com/contact)

## License
This code is distributed under the [Wowza Public License](https://github.com/WowzaMediaSystems/wse-plugin-streampublisher/blob/master/LICENSE.txt).
