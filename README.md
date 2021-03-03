# @luneo7/react-native-ntp-sync
React Native compatible implementation of the NTP Client Protocol.

Used to ensure the time used is in sync across distributed systems. The sync is achieved by the following process:

* Fetches the time from an NTP server.
* Adjusts for network latency and transfer time
* Computes the delta between the NTP server and the system clock and stores the delta for later use.
* Uses all the stored deltas to get the average time drift from UTC.
* Allows for specifying multiple NTP servers as backups in case of network errors.
* Ability to get historical details on (un)successful syncs, errors, and raw time values

## Installation
```
npm i @luneo7/react-native-ntp-sync
```

## React Native Compatibility
`React Native Version >=0.60.0` - [react-native-udp](https://www.npmjs.com/package/react-native-udp#react-native-compatibility) for more information.

## Getting Started

1. Install the module with: `yarn add @luneo7/react-native-ntp-sync` or `npm install @luneo7/react-native-ntp-sync`.
2. Install the native required dependency: `yarn add react-native-udp` or `npm install react-native-udp`.
3. Using React Native >= 0.60, on iOS run `pod install` in **ios** directory.

## Usage

Import the module into your codebase

```javascript
import ntpClient from '@luneo7/react-native-ntp-sync';

```

Create an instance of the clock object passing in the required params. See the options section below for options that can be used.

```javascript
var options = {};

// create a new instance
var clock = ntp = new ntpSync(options);

// get the current unix timestamp
var currentTime = clock.getTime();

console.log(currentTime);

// manually sync the time
var result = await clock.syncTime();
```

## Options

The client constructor can accept the following options.  **all options are optional**

##### Basic Options

* `autoSync` (boolean) : A flag to control if the client will do automatic synchronizatin. Defaults to `true`
* `history` (+int) : The number of delta values that should be maintained and used for calculating your local time drift. Defaults to `10` if not present, zero, or supplied value is not a number. **Supplied value must be > 0**
* `servers` (array of NtpServer) `{ server: string; port: number; }` : The server used to do the NTP synchronization. Defaults to 
```
[
      { server: "time.google.com", port: 123 },
      { server: "time.cloudflare.com", port: 123 },
      { server: "0.pool.ntp.org", port: 123 },
      { server: "1.pool.ntp.org", port: 123 },
]
```
* `syncInterval` (+number) : The time (in milliseconds) between each call to an NTP server to get the latest UTC timestamp. Defaults to 5 minutes
* `syncOnCreation` (boolean) : A flag to control the NTP sync upon instantiation. Defaults to `true`. (immediate NTP server fetch attempt)
* `syncTimeout` (+number) : The timeout (in milliseconds) that will be used in every NTP syncronization . Defaults to 10 seconds

```javascript
{
  "history": 10,
  "syncOnCreation": false,
  ...
}
```

## Methods

### getHistory()

Returns an `Object` of historical details generated as *clockSync* runs. It includes several fields that can be used to determine the behavior of a running *clockSync* instance. Each call represents an individual 'snapshot' of the current *clockSync* instance. History is not updated when instance is *offline*.

#### Fields

* `currentConsecutiveErrorCount` (int) : Count of current string of errors since entering an error state (`isInErrorState === true`). Resets to `0` upon successful sync.
* `currentServer` (object) : Object containing server info of the server that will be used for the next sync. Props are:
  * `server` (string) : the NTP server name
  * `port` (int) : the NTP port
* `deltas` (array&lt;object&gt;) : This array will contain a 'rolling' list of raw time values returned from each successful NTP server sync wrapped in a simple object with the following keys: (**note:** array length is limited to `config.history`; oldest at `index 0`)
  * `dt` (+/- int) : The calculated delta (in ms) between local time and the value returned from NTP.
  * `ntp` (int) : The unix epoch-relative time (in ms) returned from the NTP server. (raw value returned from server) **note**: ```ntp + -1(dt) = local time of sync```  
* `errors` (array&lt;object&gt;) : This array will contain a 'rolling' list of any errors that have occurred during sync attempts. (**note:** array length is limited to `config.history`; oldest at `index 0`). The object contains typical fields found in JS `Error`s as well as additional information.
  * `name` (string) : JavaScript Error name
  * `message` (string) : JavaScript Error message
  * `server` (object) : The server that encountered the sync error. Same keys as `currentServer` object. (possibly different values)
  * `stack` (string) : JavaScript Error stack trace (if available)
  * `time` (int) : The **local** unix epoch-relative timestamp when error was encountered (in ms)
* `isInErrorState` (boolean) : Flag indicating if the last attempted sync was an error (`true`) Resets to `false` upon successful sync.
* `lastSyncTime` (int) : The **local** unix epoch-relative timestamp of last successful sync (in ms)
* `lastNtpTime` (int) : The **NTP** unix epoch-relative timestamp of the last successful sync (raw value returned from server)
* `lastError` (object) : The error info of the last sync error that was encountered. Object keys are same as objects in the `errors` array.
* `lifetimeErrorCount` (int) : A running total of all errors encountered since *clockSync* instance was created.
* `maxConsecutiveErrorCount` (int) : Greatest number of errors in a single error state (before a successful sync).

### getTime()

Returns unix timestamp based on delta values between server and your local time. This is the time that can be used instead of ```new Date().getTime()```

### syncTime()

An on-demand method that will force a sync with an NTP server.
**NOTE:** You generally do not need to invoke a manual sync since *ntpClient* automatically runs sync according to the specified `syncInterval` interval (or its default).

### startAutoSync()

An on-demand method that will start auto sync according to the specified `syncInterval` interval (or its default). 

### stopAutoSync()

An on-demand method that will stop auto sync according.


## Based On
@jet/react0native-ntp-client is based on [react-native-clock-sync](https://github.com/artem-russkikh/react-native-clock-sync) and [react-native-ntp-client](https://github.com/artem-russkikh/react-native-ntp-client) by [Artem Russkikh](https://github.com/artem-russkikh)

## Dependencies
- [buffer](https://www.npmjs.com/package/buffer)
- [react-native-udp](https://www.npmjs.com/package/react-native-udp)