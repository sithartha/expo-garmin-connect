# expo-garmin-connect

This is a fork of https://github.com/Pythe1337N/garmin-connect to use with Expo/React-Native

---

A powerful JavaScript library for connecting to Garmin Connect for sending and receiving health and workout data. It comes with some predefined methods to get and set different kinds of data for your Garmin account, but also have the possibility to make [custom requests](#custom-requests) `GET`, `POST` and `PUT` are currently supported. This makes it easy to implement whatever may be missing to suite your needs.

## How to install

```shell
$ npm install expo-garmin-connect
```

## How to use

```js
const { GarminConnect } = require('garmin-connect');
// Create a new Garmin Connect Client
const GCClient = new GarminConnect({
    username: 'my.email@example.com',
    password: 'MySecretPassword'
});
// Uses credentials from garmin.config.json or uses supplied params
await GCClient.login();
const userProfile = await GCClient.getUserProfile();
```

Now you can check `userProfile.userName` to verify that your login was successful.

## Reusing your session

### Just save your token to db or other storage.

```js
const oauth1 = GCClient.client.oauth1Token;
const oauth2 = GCClient.client.oauth2Token;
// save to db or other storage
...
```

Reuse token:

```js
GCClient.loadToken(oauth1, oauth2);
```

### Login fallback

To make sure to use a stored session if possible, but fallback to regular login, one can use the `restoreOrLogin` method.
The arguments `username` and `password` are both optional and the regular `.login()` will be
called if session restore fails.

```js
await GCClient.restoreOrLogin(session, username, password);
```

## Events

-   `sessionChange` will trigger on a change in the current `sessionJson`

To attach a listener to an event, use the `.on()` method.

```js
GCClient.on('sessionChange', (session) => console.log(session));
```

There's currently no way of removing listeners.

## Reading data

### `getActivities(start: number, limit: number, activityType?: ActivityType, subActivityType?: ActivitySubType): Promise<IActivity[]>`

Retrieves a list of activities based on specified parameters.

#### Parameters:

-   `start` (number, optonal): Index to start fetching activities.
-   `limit` (number, optonal): Number of activities to retrieve.
-   `activityType` (ActivityType, optional): Type of activity (if specified, start must be null).
-   `subActivityType` (ActivitySubType, optional): Subtype of activity (if specified, start must be null).

#### Returns:

-   `Promise<IActivity[]>`: A Promise that resolves to an array of activities.

#### Example:

```js
const activities = await GCClient.getActivities(
    0,
    10,
    ActivityType.Running,
    ActivitySubType.Outdoor
);
```

### `getActivity(activity: { activityId: GCActivityId }): Promise<IActivity>`

Retrieves details for a specific activity based on the provided `activityId`.

#### Parameters:

-   `activity` (object): An object containing the `activityId` property.

    -   `activityId` (GCActivityId): Identifier for the desired activity.

#### Returns:

-   `Promise<IActivity>`: A Promise that resolves to the details of the specified activity.

#### Example:

```js
const activityDetails = await GCClient.getActivity({
    activityId: 'exampleActivityId'
});
```

### Download original activity data

Use the activityId to download the original activity data. Usually this is supplied as a .zip file.

```js
const [activity] = await GCClient.getActivities(0, 1);
// Directory path is optional and defaults to the current working directory.
// Downloads filename will be supplied by Garmin.
GCClient.downloadOriginalActivityData(activity, './some/path/that/exists');
```

## Custom requests

This library will handle custom requests to your active Garmin Connect session. There are a lot of different url's that is used, which means that this library probably wont cover them all. By using the network analyze tool you can find url's that are used by Garmin Connect to fetch data.

Let's assume I found a `GET` requests to the following url:

```
https://connect.garmin.com/modern/proxy/wellness-service/wellness/dailyHeartRate/22f5f84c-de9d-4ad6-97f2-201097b3b983?date=2020-03-24
```

The request can be sent using `GCClient` by running

```js
// You can get your displayName by using the getUserInfo method;
const displayName = '22f5f84c-de9d-4ad6-97f2-201097b3b983';
const url =
    'https://connect.garmin.com/modern/proxy/wellness-service/wellness/dailyHeartRate/';
const dateString = '2020-03-24';
GCClient.get(url + displayName, { date: dateString });
```

and will net you the same result as using the provided way

```js
GCClient.getHeartRate();
```

Notice how the client will keep track of the url's, your user information as well as keeping the session alive.

## Limitations

Many responses from Garmin Connect are missing type definitions and defaults to `unknown`. Feel free to add types by opening a pull request.

For now, this library only supports the following:

-   Get user info
-   Get social user info
-   Get heart rate
-   Set body weight
-   Get list of workouts
-   Add new workouts
-   Add workouts to you calendar
-   Remove previously added workouts
-   Get list of activities
-   Get details about one specific activity
-   Get the step count
-   Get earned badges
-   Get available badges
-   Get details about one specific badge
