
+++ 
title = "Accessibility and HTMX" 
date = 2023-07-23 
[taxonomies] 
tag = ["posts"] 
+++

Accessibility is hard, but it's important to get it right.  Using htmx, it's easy to make accessible web applications. It's also possible to make very inaccessible apps, too.  At its core, htmx is simply a new way to use [REST API principles](https://htmx.org/essays/how-did-rest-come-to-mean-the-opposite-of-rest/) in your browser, using [whatever server code you like](https://htmx.org/essays/hypermedia-on-whatever-youd-like/).  It doesn't generate any HTML for you, so the results are up to you and your server. 

This document lays out some guidelines to follow (and some pitfalls to avoid) on your way to building amazing and accessible apps using htmx.  

You'll need to understand the basics of web accessibility first, so please start with one of the many [resources for web accessibility](https://www.w3.org/WAI/tutorials/) that have already been published.  The same goes for basic htmx, which is simple enough to [learn in an afternoon](https://htmx.org/docs/).

## Guidelines for Accessible HTMX Applications

For the most part, web browsers have fantastic usability and accessibility features, and well-structured HTML is already accessible to a variety of adaptive technologies, like screen readers, alternate input devices, and so on.  So, if your htmx application uses standard HTML elements in standard ways, you've already done most of the work to make it accessible to people with disabilities.

### Accessibility Pitfalls for HTMX

![](https://i.kym-cdn.com/entries/icons/original/000/040/653/goldblum-quote.jpeg "A meme from the movie Jurassic Park, with the quote: Your scientists were so preoccupied with whether or not they could... they didn't stop to think if they should.")

But browsers often run into trouble when developers create complex applications that step outside the basic assumptions of the semantic web.  And htmx makes this incredibly easy to do:  

* Using custom attributes like [`hx-get`](https://htmx.org/attributes/hx-get/) and [`hx-post`](https://htmx.org/attributes/hx-post/), you can make any element on a page clickable, even if that interactivity is not announced via a screen reader.  

* With [`hx-trigger`](https://htmx.org/attributes/hx-trigger/) you can listen to any event in the system, including non-standard events, making some actions in your application unreachable to someone who needs to navigate your app using only a keyboard.

* And with [`hx-target`](https://htmx.org/attributes/hx-target/) you can update the DOM of an existing page without notifying a screen reader that anything has changed.

These missteps are easy to avoid, and following a few simple rules of thumb will actually help improve the structure of your web apps and make them more usable for non-disabled users in the process.

### 1. Native Elements and ARIA Roles 

The most common issue with accessibility in htmx applications is placing interactive attributes on elements that are not expected to be interactive, such as emulating a link using a `div` or `span`.

```html
<!-- Use native elements whenever possible -->
<button hx-get="/more-content" hx-target="#somewhere-else">Click Me Instead</button>

<!-- Use ARIA roles when you can't use native elements -->
<span role="button" tabindex="0" hx-get="/more-content" hx-target="#somewhere-else">Click Me</span>

<!-- Don't do this. It's invisible to screen readers -->
<span hx-get="/more-content" hx-target="#somewhere-else">Click Me</span>

```

For most htmx applications, there are only a few ARIA roles that you'll need to focus on.  However, make sure you're familiar with the [complete list of ARIA roles](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles) and use them whenever you can't use a native HTML element.

| Aria Role | Description |
| --- | --- |
| [`link`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/link_role) | used whenever an element will navigate users to a new page, or something that *feels* like a new page, as htmx swaps often do. |
| [`button`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Roles/button_role) | used whenever an element performs an action on the current page, such as submitting a form, opening a dialog, or inserting a new record. Essentially, if it's not a `link`, it's probably a `button`.  |


### 2. Make Elements Focusable with tabIndex

One reason that native elements are generally recommended instead of `span` and `div` is that `span` and `div` do not automatically receive focus when the user TABs through the document. For a person using the keyboard to navigate your app, it means that they may know that an action is available to them, but will be unable to reach it without a mouse.

Fortunately, HTML has a simple fix in the [tabIndex attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/tabindex), which helps developers to specify which elements can receive focus.  But this only handles half of the issue, because even when using tabIndex, users are unable to actually click on a custom element using the keyboard

Using htmx, however, the attribute `hx-trigger="click, keypress[key=='Enter']"` will mimic the default behavior of native elements, enabling the enter key to be used as a substitute for mouse clicks.  This can be tedious to add to every clickable element, so check out the [htmx accessibility extension](https://github.com/EmissarySocial/emissary/blob/main/_embed/templates/theme-global/javascript/a11y.js) (described below) which automatically applies this behavior to every element on your page.


### 3. Announce DOM Changes with aria-live

One more frustrating issue is that changes to the DOM are not automatically noticed by screen readers.  Since this is one of the fundamental operations in htmx (or any front-end library, really) it's important to take extra steps to get this right.

For most applications, the [`aria-live`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-live) attribute should be used as counterpart to every [`hx-target`](https://htmx.org/attributes/hx-target/) attribute.  This allows htmx to tell screen readers that a particular part of the DOM tree is likely to be updated in the page's lifetime.  Take care to use the appropriate settings to minimize screen reader 
interruptions.

| value | description |
| --- | --- |
| `aria-live="assertive"` | Indicates that updates to the region have the highest priority and should be presented to the user immediately |
| `aria-live="polite"` | Indicates that updates to the region should be presented at the next graceful opportunity, such as at the end of speaking the current sentence or when the user pauses typing. |
| `aria-live="off"` | (default) Indicates that updates to the region should not be presented to the user unless the user is currently focused on that region. |


### 4. Label and Describe Elements

This section is not specific to htmx, but is an important reminder for every accessible application.  For sighted users, there is a natural connection between an onscreen widget (such as an "Add" button) and its text description, usually based on how these two DOM elements are presented.  But screen readers need additional meta data in order to make this connection clear to blind users.  

The [`aria-labelledby`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-labelledby) and [`aria-describedby`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-describedby) attributes fill this need, and should be used liberally whenever possible.

In the near future, the [`aria-description`](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA/Attributes/aria-description) attribute may make this a little less cumbersome for developers, but as of July 2023, it is only a part of the draft Aria 1.3 specification, and [not supported by al browsers](https://caniuse.com/?search=aria-description).

Unfortunately, this connection cannot be backfilled, and must be coded into your applications by you.  Though it has no relation to htmx specifically, don't miss this important part of your app's accessibility.

## HTMX Power Tools for Easier Accessibility

As mentioned above, accessibility is hard, and this is especially true when your application has to redefine native HTML elements because of real-world business requirements.  Fortunately, htmx has some powerful mechanisms that will help you smooth over some of the rough edges.

You can use [htmx extensions](https://htmx.org/extensions/) to fill in the gaps that developers commonly miss when building an accessible web page.

The [htmx accessibility extension](https://github.com/EmissarySocial/emissary/blob/main/_embed/templates/theme-global/javascript/a11y.js) is one such example.  It adds missing attributes like `tabIndex` and keyboard event handlers (for ENTER key activation of links) to all HTML content on your pages, whether it was loaded at startup or dynamically via htmx.

However, it's important to remember, that there are no silver bullets, and nothing can replace diligent testing to ensure that your application is accessible to screen readers and can be navigated by a keyboard.

## Suggestions for Testing 

This section is also not specific to htmx, but is a starting point for htmx developers to test the accessibility of their apps.  There are numerous, high quality online resources that will help you test your app's accessibility, published by expert sources like: [Harvard University's Accessibility Site](https://accessibility.huit.harvard.edu/testing), the
[US Government Section 508 Website](https://www.section508.gov/test/web-software/), and the
[W3C Web Accessibility Initiative](https://www.w3.org/WAI/test-evaluate/).  In general, their recommendations break down into three categories:

### 1. Keyboard Testing

At a minimum, try to navigate your htmx application using a keyboard only.  You should be able to perform every action using only TAB, ENTER, SPACE, and ESC. You can get more details from web resources like the [Accessibility Developer Guide](https://www.accessibility-developer-guide.com/knowledge/keyboard-only/browsing-websites/), [UserWay.org](https://userway.org/blog/the-basics-of-keyboard-navigation/), or [Practical Ecommerce](https://www.practicalecommerce.com/Using-Keyboard-only-Navigation-for-Web-Accessibility).

### 2. Listen with a Screen Reader

Like proofreading a term paper, it's important to see how your htmx application performs using an actual screen reader.  This can be slow and tedious, but it will give you a good appreciation for the extra work that blind users (or even low-vision users) will have to do to use your app.

Screen readers are quirky, and their performance can vary depending on your users' computing platform.  But, there are free screen readers you can install for both [MacOS (VoiceOver)](https://accessibility.huit.harvard.edu/voiceover) and [Windows (NVDA)](https://accessibility.huit.harvard.edu/nvda), so there's no reason not to give it a try.  The results will surprise you.

### 3. Scan with Automated Tools

To be thorough, you may also want to invest your time in one or more automated scanning tool.  There are both free and commercial tools with a wide range of performance.  Your mileage may vary.

To get you started, the Harvard Accessibility Website includes an exceptionally helpful list of [free accessibility scanning tools](https://accessibility.huit.harvard.edu/auto-tools-testing#free) that you can use to evaluate the accessibility of your web app.  Like all automated tools, they have a place in your development process, but you should not rely on them for 100% of your testing.  Automated tools miss things that are obvious to people, and may also return false positives, too.  But, they are a good "catch all" to look for simple things that are easy for humans to miss.


## Accessibility Resources

### Resources

* [W3C - Designing for Web Accessibility](https://www.w3.org/WAI/tips/designing/)
* [W3C - Developing for Web Accessibility](https://www.w3.org/WAI/tips/developing/)
* [W3C - WAI Tutorials](https://www.w3.org/WAI/tutorials/)
* [Harvard Digital Accessibility Resources](https://accessibility.huit.harvard.edu)

### Specifications

* [W3C Accessibility Guidelines (WCAG)](https://www.w3.org/TR/wcag-3.0/)
* [Accessible Rich Internet Applications (WAI-ARIA)](https://www.w3.org/TR/wai-aria-1.2/)
* [Mozilla MDN Resources](https://developer.mozilla.org/en-US/docs/Web/Accessibility/)


## One More Thing... Rule 7b

There's a running joke on the [htmx Discord server](https://discord.com/channels/725789699527933952/725789747212976259), about "Rule 7b" which doesn't exist, of course.  But if it did, would say something like: "if you propose an idea, you're on the hook for implementing it."  Or in other words, please help us make htmx better with your actions, and not just your opinions.

Like any software, htmx can use your help and expertise in countless ways, and our community is always open to new ideas of how we can improve.  If you, dear reader, see ways that this tutorial, or htmx itself can do a better job at helping developers make accessible websites, please help us out by [opening an issue or GitHub](https://github.com/bigskysoftware/htmx/issues), or better yet by [submitting a Pull Request](https://github.com/bigskysoftware/htmx/pulls) with helpful fixes.

🐝🐝🐝🐝🐝🐝🐝