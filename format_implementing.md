# Format Implementation Guide

If you're a launcher developer, this document can help you by providing useful tips and recommended practices for implementing the MODIP Format.

---

## Rendering HTML

The MODIP Format Specification allows the usage of HTML in certain fields, such as `description` and `changelog`. HTML is widely used and meets many of the requirements needed for a field like `description`. However, HTML can be **very dangerous**. Take caution when displaying content using HTML. It's recommended to sanitize your HTML according to the rules listed below.

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

---

## Implementing Frameworks

Frameworks also exist as MODIP Format Projects. It's important for launchers to understand the Project metadata and install frameworks as needed.

Framework IDs MUST be prefixed with `framework-`, so launchers can tell if a project is a Framework.
For frameworks with the installation method set to `versionJsonInstall`, launchers should read the version JSON metadata specified in a file with `rel` set to `versionJson`. An example of a modloader using `versionJsonInstall` would be Fabric.

Launchers may deviate from or completely ignore the value set in the installation method for the version.

The `runForgeInstaller` installation method exists to tell launchers that this contains a Forge installer. Launchers may choose to use their own method of installation if they would like.

---

## Implementing Dependencies

When a user downloads a project using a launcher, it's important to download its required dependencies. Launchers should combine the version dependencies with the file dependencies in order to create a full dependency list.

For the purposes of this section, the term "the host" refers to the service a launcher is using in order to download projects, such as Diluv or Modrinth.

If a dependency's ID is `minecraft`, then it's the required Minecraft version. Implementation of where to obtain Minecraft metadata is up to the developer of a launcher. Launchers may download metadata from the host, from Mojang directly, or from another service.

If a dependency's ID is not `minecraft`, then it's a required project.

If a dependency has the `src` field, it means that at that specified URL is full metadata about this dependency. Launchers should check here first if it's specified. If the URL does not serve a suitable result, or there is no `src` field, launchers should then ask the host for a project matching the same ID. If at this point the launcher still hasn't found any metadata for the project, the launcher may choose to ask another host that it knows of for a project with the same ID, or it may choose to fail and warn the user.

---

## Implementing Conditions

Conditions allow fields to be changed based on certain variables. Conditions can be present on almost any field, so it's important to know what is or isn't a condition.

Fields which are an Object cannot be a condition themselves. For example, the `license` field cannot be a conditions object.

When parsing an object with conditions, apply conditions at the end. For example, consider the following object:

```json
"conditions": {
  "environment": {
    "client": {
      "required": false
    },
    "server": {
      "required": true
    }
  }
},
"required": true
```

If your launchers considers itself to be a client, the value of `required` should be false, even though conditions are listed before the `"required": true` value.
