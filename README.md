# CMP `<dialog>` top-layer overlay demo

A small static page that demonstrates a native `<dialog>` stacking conflict and a possible fix.

When a CMP is rendered as a native modal `<dialog>` (`dialogMode`), it lives in the browser **top layer**.
The top layer stacks modal dialogs in **open order** — the last `showModal()` call is always painted on top,
regardless of `z-index`. So a site dialog opened via `showModal()` *after* the CMP can cover it.

## Usage

1. Open the page and wait for the consent dialog.
2. Click **Open competing modal** → it covers the CMP (the issue).
3. Tick **Apply fix**, reload, repeat → the competing dialog is downgraded to non-modal `show()` and no
   longer covers the CMP.

## The fix

While the CMP is open, other dialogs' `showModal()` is redirected to non-modal `show()` so the CMP keeps the
top layer:

```js
const original = HTMLDialogElement.prototype.showModal;
HTMLDialogElement.prototype.showModal = function () {
  if (isCmpDialog(this)) return original.call(this); // allow the CMP's own dialog
  return HTMLDialogElement.prototype.show.call(this); // downgrade everyone else
};
```

> Trade-off: this downgrades **all** other modal dialogs while the CMP is open.

## Query params

| Param | Default | Purpose |
|---|---|---|
| `settingsId` | `TxBnOLo9B` | settings id (a TCF test config) |
| `country` | `DE` | forced country so the dialog always shows |
| `bundle` | prod `latest` | bundle URL (point at a custom build to test a patched bundle) |
| `fix` | _(off)_ | `?fix=1` applies the fix on load |
| `autoOpen` | `0` | `?autoOpen=2000` auto-opens the competing modal after N ms |
