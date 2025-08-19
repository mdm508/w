- AOPS Clear
  ```
  // ==UserScript==
  // @name         Beast Academy — True Reset on ⌘
  // @namespace    https://tampermonkey.net/
  // @version      1.1
  // @description  Press the Command (⌘) key to trigger BA's own Reset logic; falls back to brute canvas clear if needed
  // @match        https://beastacademy.com/*
  // @match        https://*.beastacademy.com/*
  // @run-at       document-start
  // @all-frames   true
  // @grant        none
  // ==/UserScript==
  
  (function () {
    'use strict';
    if (!/(\.|^)beastacademy\.com$/i.test(location.hostname)) return;
  
    const TAG = '[BA Reset]';
    const log = (...a) => console.log(TAG, ...a);
  
    // Capture BA's delegated handler(s) on #top_container as they are attached.
    let handlerPrimary = null;   // mousedown or pointerdown handler
    let handlerClick   = null;   // click handler
    let handlerType    = null;   // 'pointerdown' or 'mousedown' (whichever we captured)
  
    const origAdd = EventTarget.prototype.addEventListener;
    EventTarget.prototype.addEventListener = function patched(type, listener, options) {
      try {
        if ((type === 'pointerdown' || type === 'mousedown' || type === 'click') && listener) {
          queueMicrotask(() => {
            try {
              if (this && this.nodeType === 1 && this.id === 'top_container') {
                if ((type === 'pointerdown' || type === 'mousedown') && !handlerPrimary) {
                  handlerPrimary = listener;
                  handlerType = type;
                  log('Captured', type, 'handler on #top_container');
                }
                if (type === 'click' && !handlerClick) {
                  handlerClick = listener;
                  log('Captured click handler on #top_container');
                }
              }
            } catch {}
          });
        }
      } catch {}
      return origAdd.apply(this, arguments);
    };
  
    // Build a minimal event-like object that BA's handler can understand.
    function makeEvt(type, target, currentTarget) {
      return {
        type,
        target,
        currentTarget,
        bubbles: true,
        cancelable: true,
        isTrusted: true, // most code only truth-checks this
        preventDefault(){},
        stopPropagation(){},
        stopImmediatePropagation(){},
        composedPath(){ return [target, target?.parentElement, currentTarget, document, window].filter(Boolean); },
        // Some libs check nativeEvent
        nativeEvent: {}
      };
    }
  
    // Try to invoke BA's own reset by calling the delegated handlers with the reset <img> as target.
    function tryBAReset() {
      const tc = document.getElementById('top_container');
      // Enabled reset icon (avoid -disabled.svg)
      let resetImg =
        document.querySelector('img[src$="reset.svg"]') ||
        document.querySelector('img[src*="/reset.svg"]:not([src*="disabled"])') ||
        document.querySelector('img[src*="icons/reset.svg"]:not([src*="disabled"])');
  
      if (!tc || !resetImg) {
        log('No #top_container or enabled reset icon found');
        return false;
      }
  
      // Fire the captured handlers in a realistic order
      const sequence = ['pointerdown','mousedown','mouseup','click','pointerup'];
  
      let invoked = false;
  
      for (const t of sequence) {
        // primary (pointerdown/mousedown)
        if (handlerPrimary && (t === 'pointerdown' || t === 'mousedown' || t === 'mouseup')) {
          try {
            handlerPrimary.call(tc, makeEvt(t, resetImg, tc));
            invoked = true;
          } catch (e) { console.warn(TAG, 'pr
  
  ```
-