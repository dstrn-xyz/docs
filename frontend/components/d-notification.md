# d-notification

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/notification.mp4" type="video/mp4">
  </video>
</p>

toast notification component with linear progress bar countdown and automated exit lifecycle.

```html
<d-notification timer="4000" opened>
  <p>project settings saved successfully</p>
</d-notification>
```

## attributes and properties

| attribute | property | type      | default | form | description                                                  |
| :-------- | :------- | :-------- | :------ | :--- | :----------------------------------------------------------- |
| `opened`  | `opened` | `boolean` | `false` | no   | whether the toast notification is displayed                  |
| `timer`   | `timer`  | `number`  | `0`     | no   | auto dismissal duration in milliseconds (`0` disables timer) |

## programmatic api

```javascript
const toast = select('d-notification');

// show toast and start progress bar countdown
toast.show();

// hide toast
toast.hide();

// hide, wait for animation to finish, and remove from dom
await toast.destroy();
```

## events

| event   | detail | bubbles | description                         |
| :------ | :----- | :------ | :---------------------------------- |
| `open`  | none   | yes     | dispatched when notification opens  |
| `close` | none   | yes     | dispatched when notification closes |

## behavior

- displays toast notifications with an optional countdown progress bar that automatically dismisses after the timer expires.
