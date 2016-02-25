# Siesta Example Project

This app allows you to type a Github username and see the user’s name, avator, and repos.

This is a simple app, and intentionally minimizes things outside of Siesta’s purview: no models, minimal functionality, and bare bones UI. (Well, there is the gratuitous use of the Siesta color scheme!)

## What’s interesting about it?

The app does a live search as you type. The user’s repo list comes from a URL provided in the user profile, so each keystroke triggers a two-request chain.

This cascade of API requests and responses poses several problems:

- **Race condition:** There’s no guarantee that responses arrive in the order requests were sent. If you type `AB`, the app sends requests for `/users/A`, `/users/A/repos`, `/users/AB`, `/users/AB/repos`. If the app naively populates the UI with whatever response arrives last, the UI could for example end up showing user AB’s profile but user A’s respositories. Instead, we have to make sure that the received response corresponds to the thing the UI wants to show. This is a tricky — or at the very least annoying — problem to solve with standard old response callbacks.
- **Redundant requests:** We don’t want the app to re-request a username that the user already typed. In theory `NSURLCache`, solves this; in practice, you’ll end up fighting hard with the server response headers and the cache’s settings to get the behavior you want.
- **Unnecessary requests:** As the user types, we don’t want to fill the request pipeline with requests whose results we’ll never use. What want want is to cancel a request after a short delay — but only if the request is no longer needed! A rapid backspace shouldn’t cause a double request.
- **Wiping cache and UI on logout:** We don’t want bits of sensitive information lingering in some view after the user has logged out.

Siesta solves all these problems transparently, with minimal code.

## Files of note

- `Source/API/GithubAPI.swift` shows how to:
    
    - set up a Siesta service,
    - send an authentication header, and
    - add a custom response transformers that:
        - wrap all JSON responses with SwiftJSON,
        - map endpoints to models, and
        - replace Siesta’s default error messages with Github-provided messages when present.

- `Source/UI/UserViewController.swift` shows how to:
    
    - use Siesta to propagate changes from a Resource to a UI,
    - retarget a view controller at different Resources while it is visible,
    - use `ResourceStatusOverlay` to show a spinner and default error message, and
    - use Siesta’s caching, throttling, and delayed cancellation to manage a rapid series of requests triggered by keystrokes.

- `Source/UI/RepositoryListViewController.swift` shows how to:
    
    - create a view controller which displays a Siesta resource determined by a parent VC and
    - populate a table view with Siesta.

## Rate limit errors?

If you hit the Github API’s rate limit while running the demo, press the “Log In” button. If you’re experimenting with the demo a lot, you can set `GITHUB_USER` and `GITHUB_PASS` environment variables in the “Run” build scheme to make the app automatically log you in on launch.

You can use a [personal access token](https://github.com/settings/tokens) in place of your password. You don’t need to grant any permissions to your token for this app; just the public access will do.