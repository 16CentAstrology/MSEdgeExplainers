# Gamepad Trigger-Rumble

Authors: [Gabriel Brito](https://github.com/gabrielsanbrito), [Steve Becker](https://github.com/SteveBeckerMSFT), [Ben Mathwig](https://github.com/bmathwig)

## Status of this Document

This document is a starting point for engaging the community and standards bodies in developing collaborative solutions fit for standardization. As the solutions to the problems described in this document progress along the standards-track, we will retain this document as an archive and use this section to keep the community up-to-date with the most current standards venue and content location of future work and discussions.

- This document status: **Archived**
- Expected venue: [W3C Web Applications Working Group](https://www.w3.org/2019/webapps/)
- Current version: **This document**

## Definitions

### Dual-Rumble

It is the haptic feedback effect generated by two Eccentric Rotating Mass (ERM) actuators, one in each grip of the gamepad.

### Trigger-Rumble

It is the haptic feedback effect generated by two independent motors, with one motor located in each of the gamepad's triggers.

## Introduction

Currently, there are gamepads, like the [Microsoft's Xbox](https://www.xbox.com/en-us/accessories/controllers/xbox-wireless-controller#white) and the [Razer Wolverine Ultimate](https://www.razer.com/console-controllers/razer-wolverine-ultimate/RZ06-02250100-R3U1) Controllers, which comes equipped with "impulse triggers" capable of providing trigger-rumble feedback to the user. While the [GamepadHapticActuator](https://w3c.github.io/gamepad/extensions.html#gamepadhapticactuator-interface), part of the [Gamepad Haptics Extension](https://w3c.github.io/gamepad/extensions.html) proposal, already provides support for dual-rumble effect, it does not allow developers to have access to trigger-rumble yet.

This change will allow game developers to equip their web applications with a wider set of haptics feedback options, making it possible for their users to have a richer and more engaging experience on the web.

## Goals

This explainer proposes an extension to the GamepadHapticActuator interface to expose the trigger-rumble capability in the Web for compatible gamepads.

## Proposed Solution

A second type of gamepad haptic effect called "trigger-rumble" should be defined to represent the feedback generated by the two independent trigger motors. Therefore, we should add a "trigger-rumble" entry to the enum GamepadHapticEffectType.

```js
enum GamepadHapticEffectType {
    "dual-rumble",
    "trigger-rumble"
};
```

Moreover, it is necessary to add to the GamepadEffectParameters dictionary the double values "LeftTrigger" and "RightTrigger" to represent the trigger motors' rumble intensity normalized to the interval [0.0, 1.0].

```js
dictionary GamepadEffectParameters {
    double duration;
    double startDelay;
    double strongMagnitude;
    double weakMagnitude;
    double leftTrigger;
    double rightTrigger;
};
```

Since not every gamepad has the trigger-rumble feature, the developer must check the `effects` array to query the device's capabilities. This array should contain all the `GamepadHapticEffectType` strings that the gamepad device supports. We propose implementing an array like this, because it also allow applications to detect and discover new types of haptics effects - please refer to the [Alternative Solutions](#alternative-solutions) section for more info on other options considered. It should be noted that devices with trigger-rumble support should also be capable to provide dual-rumble feedback.

```js
interface GamepadHapticActuator {
    [SameObject] readonly attribute FrozenArray<GamepadHapticEffectType> effects;
};
```


## Use Cases

The trigger-rumble effect could be activated for both triggers of a gamepad at index 0 using the code below (1 second, 50% intensity on left trigger and full on right), given that the device has support for it. Moreover, if the device is not able to play trigger rumble feedback, it should be capable to fallback to dual rumble.

```js
let gamepads = navigator.getGamepads();
if (gamepads.length > 0) {
  let gamepad = gamepads[0];
  if (gamepad.vibrationActuator) {
    if (gamepad.vibrationActuator.effects && gamepad.vibrationActuator.effects.includes("trigger-rumble")) {
      gamepad.vibrationActuator.playEffect("trigger-rumble", {
        duration: 1000,
        leftTrigger: 0.5,
        rightTrigger: 1.0,
      });
    } else {
      gamepad.vibrationActuator.playEffect("dual-rumble", {
        duration: 1000,
        strongMagnitude: 0.5,
        weakMagnitude: 1.0,
      });
    }
  }
}
```

Also, it is possible to activate the feedback only for one trigger by omitting the other one.

## Privacy Considerations

Allowing websites to query for Gamepads' haptic capabilities might raise fingerprinting concerns. However, `navigator.getGamepads` method returns an empty list before a gamepad user gesture has been detected (https://www.w3.org/TR/gamepad/#getgamepads-method), which will not allow queries to be done before the user has interacted with the gamepad in the website. 

## Alternative Solutions

### `GamepadHapticActuatorType` entry for effect support detection
Adding a new entry entry, e.g. "trigger-rumble" to GamepadHapticActuatorType might be difficult to accomplish without breaking existing apps. For example, an app that always checks for "dual-rumble" may break if Xbox controllers suddenly begin to return "trigger rumble" as their `GamepadHapticActuatorType`. Therefore, it may be better to let the web applications do the required capability detection using a different route - e.g., by checking the `effects` array -; and always define `GamepadHapticActuator.type` to be "dual-rumble", so apps cannot rely on it, then deprecate, and eventually remove it.

### `canPlay` method
A new `canPlay` method that takes a `GamepadHapticEffectType` string and returns a `boolean` could also be used to query a gamepad's haptics capabilities. However, this requires a new call to be made for every `GamepadHapticEffectType` we would like to check for support. Moreover, implementing a method like this could affect legacy engines, in case effect types need to be deprecated or renamed.

## Future Plans for this API
The extension to the `GamepadHapticsActuator` is a short-term solution to unblock developers from being limited to 2-motor rumble on current generation hardware. In the long-term, we plan to invest in the [HapticsDevice API](https://github.com/MicrosoftEdge/MSEdgeExplainers/blob/main/HapticsDevice/explainer.md), which will supercede this API, the [Vibration API](https://w3c.github.io/vibration/#dom-navigator-vibrate), and will generalize haptics behaviors across many devices and applications. This new `HapticsDevice` API will be applied to Gamepad to support rumble and haptic effects in previous and next generation hardware.

## References

[Gamepad Haptics API proposal](https://docs.google.com/document/d/1jPKzVRNzzU4dUsvLpSXm1VXPQZ8FP-0lKMT-R_p-s6g/edit#)  
[Microsoft Xbox Controller](https://www.xbox.com/en-us/accessories/controllers/xbox-wireless-controller#white)  
[Razer Wolverine Ultimate Controller](https://www.razer.com/console-controllers/razer-wolverine-ultimate/RZ06-02250100-R3U1)  
[W3C Gamepad Working Draft](https://www.w3.org/TR/gamepad/#getgamepads-method)  
[W3C Gamepad Extensions draft](https://w3c.github.io/gamepad/extensions)  
[W3C Gamepad issue 138 - Xbox One impulse trigger effects](https://github.com/w3c/gamepad/issues/138)
