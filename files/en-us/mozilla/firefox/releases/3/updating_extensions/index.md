---
title: Updating extensions for Firefox 3
slug: Mozilla/Firefox/Releases/3/Updating_extensions
page-type: guide
sidebar: firefox
---

This article provides information that will be useful to developers that wish to update their extensions to work properly under Firefox 3.

Before going further, there's one helpful hint we can offer: if the only change your extension requires is a bump to the `maxVersion` field in its install manifest, and you host your extension at [addons.mozilla.org](https://addons.mozilla.org/), you don't actually need to upload a new version of your extension! Use the Developer Control Panel at AMO to adjust the `maxVersion`. You can avoid having to have your extension re-reviewed this way.

## Step 1: Update the install manifest

The first step — and, for most extensions, the only one that will be needed — is to update the [install manifest](https://web.archive.org/web/20210421140209/https://developer.mozilla.org/en-US/docs/Archive/Add-ons/Install_Manifests) file, [`install.rdf`](https://web.archive.org/web/20160809001138/https://developer.mozilla.org/en-US/Add-ons/Themes/Obsolete/Creating_a_Skin_for_Firefox/install.rdf), to indicate compatibility with Firefox 3.

Find the line indicating the maximum compatible version of Firefox (which, for Firefox 2, might look like this):

```xml
<em:maxVersion>2.0.*</em:maxVersion>
```

Change it to indicate compatibility with Firefox 3:

```xml
<em:maxVersion>3.0.*</em:maxVersion>
```

Then reinstall your extension.

Note that Firefox 3 does away with the extra ".0" in the version number, so instead of using `3.0.0.*`, you only need to use `3.0.*`.

There have been (and will continue to be) a number of API changes that will likely break some extensions. We're still working on compiling a complete list of these changes.

> [!NOTE]
> If your extension still uses an [`Install.js`](https://web.archive.org/web/20210604075726/https://developer.mozilla.org/en-US/docs/Archive/Install.js) script instead of an [install manifest](https://web.archive.org/web/20210421140209/https://developer.mozilla.org/en-US/docs/Archive/Add-ons/Install_Manifests), you need to make the transition to an install manifest now. Firefox 3 no longer supports `install.js` scripts in XPI files.

### Add localizations to the install manifest

Firefox 3 supports new properties in the install manifest to specify localized descriptions. The old methods still work however the new allow Firefox to pick up the localizations even when the add-on is disabled and pending install. See [Localizing extension descriptions](https://web.archive.org/web/20210126131244/https://developer.mozilla.org/en-US/docs/Mozilla/Localization/Localizing_extension_descriptions) for more details.

## Step 2: Ensure you are providing secure updates

If you are hosting addons yourself and not on a secure add-on hosting provider like [addons.mozilla.org](https://addons.mozilla.org/) then you must provide a secure method of updating your add-on. This will either involve hosting your updates on an SSL website, or using cryptographic keys to sign the update information. Read [Securing Updates](https://web.archive.org/web/20201031093738/https://developer.mozilla.org/en-US/docs/Archive/Add-ons/Extension_Versioning,_Update_and_Compatibility#securing_updates) for more information.

## Step 3: Deal with changed APIs

Several APIs have been changed in significant ways. The most significant of these, which will likely affect a large number of extensions, are:

### DOM

Nodes from external documents should be cloned using [`document.importNode()`](/en-US/docs/Web/API/Document/importNode) (or adopted using [`document.adoptNode()`](/en-US/docs/Web/API/Document/adoptNode)) before they can be inserted into the current document. For more on the [`Node.ownerDocument`](/en-US/docs/Web/API/Node/ownerDocument) issues, see the [W3C DOM FAQ](https://www.w3.org/DOM/faq.html#ownerdoc).

Firefox doesn't currently enforce this rule (it did for a while during the development of Firefox 3, but too many sites break when this rule is enforced). We encourage Web developers to fix their code to follow this rule for improved future compatibility.

### Bookmarks & History

If your extension accesses bookmark or history data in any way, it will need substantial work to be compatible with Firefox 3. The old APIs for accessing this information have been replaced by the new [Places](https://web.archive.org/web/20210620103113/https://developer.mozilla.org/en-US/docs/Mozilla/Tech/Places) architecture. See the [Places migration guide](https://web.archive.org/web/20200621121524/https://developer.mozilla.org/en-US/docs/Mozilla/Tech/Places/Places_Developer_Guide) for details on updating your existing extension to use the Places API.

### Download Manager

The Download Manager API has changed slightly due to the transition from an RDF data store to using the [Storage](https://web.archive.org/web/20210401045303/https://developer.mozilla.org/en-US/docs/Mozilla/Tech/XPCOM/Storage) API. This should be a pretty easy transition to make. In addition, the API for monitoring download progress has changed to support multiple download manager listeners. See `nsIDownloadManager`, `nsIDownloadProgressListener`, and [Monitoring downloads](https://web.archive.org/web/20210516125311/https://developer.mozilla.org/en-US/docs/Archive/Mozilla/Monitoring_downloads) for more information.

### Password Manager

If your extension accesses user login information using the Password Manager, it will need to be updated to use the new Login Manager API.

- The article [Using nsILoginManager](https://web.archive.org/web/20210530180123/https://developer.mozilla.org/en-US/docs/Mozilla/Tech/XPCOM/Reference/Interface/nsILoginManager/Using_nsILoginManager) includes examples, including a demonstration of how to write your extension to work with both the Password Manager and the Login Manager, so it will work with both Firefox 3 and earlier versions.
- `nsILoginInfo`
- `nsILoginManager`

You can also override the built-in password manager storage if you want to provide your own password storage implementation in your extensions. See [Creating a Login Manager storage module](https://web.archive.org/web/20210515154057/https://developer.mozilla.org/en-US/docs/Mozilla/Creating_a_login_manager_storage_module) for details.

### Popups (Menus, Context Menus, Tooltips and Panels)

The XUL Popup system was heavily modified in Firefox 3. The Popup system includes main menus, context menus and popup panels. A guide to [using Popups](https://web.archive.org/web/20210418010207/https://developer.mozilla.org/en-US/docs/Archive/Mozilla/XUL/PopupGuide) has been created, detailing how the system works. One thing to note is that `popup.showPopup` has been deprecated in favor of new `popup.openPopup` and `popup.openPopupAtScreen`.

### Autocomplete

The `nsIAutoCompleteController` interface's `handleEnter()` method has been changed to accept an argument that indicates whether the text was selected from the autocomplete popup or by the user pressing enter after typing text.

### DOMParser

- When a `DOMParser` is instantiated, it inherits the calling code's principal and the `documentURI` and `baseURI` of the window the constructor came from.
- If the caller has UniversalXPConnect privileges, it can pass parameters to `new DOMParser()`. If fewer than three parameters are passed, the remaining parameters will default to `null`.
  - The first parameter is the principal to use; this overrides the default principal normally inherited.
  - The second parameter is the `documentURI` to use.
  - The third parameter is the `baseURI` to use.

- If you initialize a `DOMParser` using a contract, such as by calling `createInstance()`, and you don't call the `DOMParser`'s `init()` method, attempting to initiate a parsing operation will automatically create and initialize the `DOMParser` with a null principal and `null` pointers for `documentURI` and `baseURI`.

### Stop using the internal string API

The internal string API is no longer exported; you must migrate to the external string API. See these articles for helpful information:

- [Mozilla external string guide](https://web.archive.org/web/20160423162648/https://developer.mozilla.org/en-US/docs/Mozilla/Mozilla_external_string_guide)
- [XPCOM Glue](https://web.archive.org/web/20210625030032/https://developer.mozilla.org/en-US/docs/Mozilla/Tech/XPCOM/Glue)
- [Migrating from Internal Linkage to Frozen Linkage](https://web.archive.org/web/20210620000937/https://developer.mozilla.org/en-US/docs/Archive/Add-ons/Migrating_from_Internal_Linkage_to_Frozen_Linkage)

### Removed interfaces

The following interfaces were removed from Gecko 1.9, which drives Firefox 3. If your extension makes use of any of these, you'll need to update your code:

- `nsIDOMPaintListener`
- `nsIDOMScrollListener`
- `nsIDOMMutationListener`
- `nsIDOMPageTransitionListener`
- `nsICloseAllWindows` (see [Firefox bug 386200](https://bugzil.la/386200))

## Step 4: Check for relevant chrome changes

There have been a few changes to chrome layout that may affect some extensions.

### New boxes

There has been a minor change to the chrome that may require changes in your code. A new `vbox` has been added, called "browser-bottombox", which encloses the find bar and status bar at the bottom of the browser window. Although this doesn't affect the appearance of the display, it may affect your extension if it overlays chrome relative to these elements.

For example, if you previously overlaid some chrome before the status bar, like this:

```xml
<window id="main-window">
  <something insertbefore="status-bar" />
</window>
```

You should now overlay it like this:

```xml
<vbox id="browser-bottombox">
  <something insertbefore="status-bar" />
</vbox>
```

Or use the following technique to make your overlay work on both Firefox 2 and Firefox 3:

```xml
<window id="main-window">
  <vbox id="browser-bottombox" insertbefore="status-bar">
    <something insertbefore="status-bar" />
  </vbox>
</window>
```

### Changed boxes

Extensions that attempt to overlay onto the "appcontent" box try to float chrome over document content will no longer display that material. You should update your extension to use the new [`<xul:panel>`](https://web.archive.org/web/20210301150646/https://developer.mozilla.org/en-US/docs/Archive/Mozilla/XUL/panel) XUL element. If you wish to keep the panel from automatically disappearing after a delay, you can set the `noautohide` attribute to `true`.

## Other changes

_Add simple changes you had to make while updating your extension to work with Firefox 3 here._

- `chrome://browser/base/utilityOverlay.js` is no longer supported for security reasons. If you were previously using this, you should switch to `chrome://browser/content/utilityOverlay.js`.
- `nsIAboutModule` implementations are now required to support the `getURIFlags` method. See [nsIAboutModule.idl](https://searchfox.org/mozilla-central/source/netwerk/protocol/about/nsIAboutModule.idl) for documentation. This affects extensions that provide new `about:` URIs. ([Firefox bug 337746](https://bugzil.la/337746))
- The [`<xul:tabbrowser>`](https://web.archive.org/web/20210221234616/https://developer.mozilla.org/en-US/docs/Archive/Mozilla/XUL/tabbrowser) element is no longer part of "toolkit" ([Firefox bug 339964](https://bugzil.la/339964)). This means this element is no longer available to XUL applications and extensions. It continues to be used in the main Firefox window (browser.xul).
- Changes to `nsISupports_proxies` and possibly to threading-related interfaces need to be documented.
- If you use XML processing instructions, such as `<?xml-stylesheet ?>` in your XUL files, be aware of the changes made in [Firefox bug 319654](https://bugzil.la/319654):
  1. XML PIs are now added to a XUL document's DOM. This means {{ Domxref("Node.firstChild", "document.firstChild") }} is no longer guaranteed to be the root element. If you need to get the root document in your script, use {{ Domxref("document.documentElement") }} instead.
  2. `<?xml-stylesheet ?>` and `<?xul-overlay ?>` processing instructions now have no effect outside the document prolog.

- `window.addEventListener("load", myFunc, true)` is not fired when loading web content (browser page loads). This is due to [Firefox bug 296639](https://bugzil.la/296639) which changes the way inner and outer windows communicate. The simple fix here is to use `gBrowser.addEventListener("load", myFunc, true)` and works in Firefox 2 as well.
- `content.window.getSelection()` gives an object (which can be converted to a string by `toString()`), unlike the now deprecated `content.document.getSelection()` which returns a string
- `event.preventBubble()` was deprecated in Firefox 2 and has been removed in Firefox 3. Use [`event.stopPropagation()`](/en-US/docs/Web/API/Event/stopPropagation), which works in Firefox 2 as well.
- Timers that are initiated using {{domxref("Window.setTimeout", "setTimeout()")}} and {{domxref("WorkerGlobalScope.setTimeout", "setTimeout()")}} are now blocked by modal windows due to the fix made for [Firefox bug 52209](https://bugzil.la/52209). You may use `nsITimer` instead.
- If your extension needs to allow an untrusted source (e.g., a website) to access the extension's chrome, then you must use the new [`contentaccessible` flag](https://web.archive.org/web/20210623201644/https://developer.mozilla.org/en-US/docs/Mozilla/Chrome_Registration#contentaccessible).
