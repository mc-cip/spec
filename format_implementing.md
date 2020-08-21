# Format Implementation Guide

If you're a launcher developer, this document can help you by providing useful tips and recommended practices for implementing the MCIP Format.

## Rendering HTML

The MCIP Format Specification allows the usage of HTML in certain fields, such as `description` and `changelog`. HTML is widely used and meets many of the requirements needed for a field like `description`. However, HTML can be **very dangerous**. Take caution when displaying content using HTML. It's recommended to sanitize your HTML according to the rules listed below.

### Disallowed Tags

You should never attempt to render any of the following tags: `script`, `style`, `link`, `title`, `button`, `form`, `input`, `meta`, `option`, `textarea`, and `select`.

### iframes

`iframes` can be used, but it's recommended to only render them when the hostname is from a trustworthy source. Trustworthy sources are up to implementation by the launcher.

### Scripting

Forms of client side code execution, such as JavaScript and WebAssembly, _should never be allowed_. XSS attacks are dangerous - do not allow untrusted code to run in your launcher.

### Sanitization

It's recommend to whitelist tags and attributes based on conventions from a host. For example, if the host you're integrating your launcher with only uses `<p>` tags, only allow `<p>` tags to be rendered. It's a better approach that minimizes any potential risk.

Make sure to sanitize all attributes. For example, XSS attacks can use the `onerror` attribute to run JavaScript code after loading an invalid image. Only allow the attributes that are required.

### Frequently Asked Questions

**Q: I've received HTML in a field that doesn't allow for it in the specification. Should I parse it anyway?**  
A: No. Do not attempt to parse HTML outside of a field that it's allowed in. If you receive a string, `<p>My Mod</p>` in the `name` field, render it exactly as written, including the HTML tags. Don't attempt to strip HTML from a value.

**Q: I've received Markdown. Should I parse it?**  
A: Markdown is not allowed in the specification. You shouldn't attempt to parse it. Hosts should be providing HTML, not Markdown.

**Q: I don't want to render HTML at all. Should I just strip it out?**  
A: If for whatever reason you do not want to render HTML, then yes, the best method is to remove the tags. Keep in mind the consequences of not rendering HTML, as users may lose helpful and useful features such as images and video embeds.
