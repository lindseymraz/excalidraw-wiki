%% <%* throw(new Error("Cannibalization")); %> %%
# [[Templater Tricks]]
**see also**:: [[Hybrid markdown-excalidraw note template]]
You can find Obsidian Templater on [GitHub](https://github.com/SilentVoid13/Templater) and in [Community Plugins](obsidian://show-plugin?id=templater-obsidian).

## Templater Cannibalizes Templates with Obsidian Sync

This hack resolves an issue I've been struggling with for years. You can find the relevant discussions and issues here: [#1096](https://github.com/SilentVoid13/Templater/discussions/1096), [#1100](https://github.com/SilentVoid13/Templater/issues/1100).

The problem occurs when Templater is set to run on new file creation. Every time you synchronize your template file to a new device, Templater processes the template again. Depending on the features in the template, this can make the template unusable, as your code gets replaced with the processed text. For example, a line like `![[<% tp.file.title%>.svg]]` would be replaced during sync with `![[Title of my template.svg]]`. This becomes frustrating, especially when syncing your Vault across multiple devices.

My solution is to throw an error when a specific condition is detected, such as a file name or folder location. 

Add the following line of code to the start of your templates. When the file is used as intended, Templater will replace the code with an empty string. However, if Templater tries to process the template during a sync, the code will trigger an error and stop Templater from "cannibalizing" your template.

### Abort if file is in the Templater templates folder
```js
<%*
  const PATH = app.plugins.plugins["templater-obsidian"].settings.templates_folder;
  if(tp.file.folder(true) === PATH) throw(new Error("Cannibalization"));
%>
```

### Abort if the file has a specific title
```js
<%*
  if(tp.file.title === "REPLACE THIS WITH FILENAME OF YOUR TEMPLATE") {
    throw(new Error("Cannibalization"));
  }
%>
```
or as a one liner:
```js
<%* if(tp.file.title === "FILE TITLE") throw(new Error("Cannibalization"));%>
```

### To ensure a file is always avoided by Templater
A good example would be this article in my Obsidian Vault. When ever this note is synchronized to other devices the scripts in the code-blocks get processed by Templater and replaced with parsed results 😫. The same will happen if you create a note just for yourself where you collect Templater snippets and best practices. You can avoid this frustration by adding the following as the very first line of any document (right after document properties): `%% <%* throw(new Error("Cannibalization")); %> %%`. This will ensure Templater aborts any time it tries to touch your file.

## Open Excalidraw Drawing After Creation with Templater

In this video, I demonstrate how to create intelligent Excalidraw templates, such as project templates pre-filled with project data or daily note templates that include a daily quote or even a [daily quote illustration generated by Dall-e](https://youtu.be/Tr1rzThssL8). By combining Templater with Excalidraw, you can automate much of this process. However, there's an issue: when a new file is created, Obsidian initially shows the markdown view, requiring you to manually switch to the Excalidraw view. This happens because Obsidian hasn't indexed the document's properties yet, so the Excalidraw plugin doesn't recognize the new file as an Excalidraw drawing.

![Excalidraw Templates](https://youtu.be/jgUpYznHP9A)

You can solve this problem by adding a short piece of code to your template. This code adds an event listener to Obsidian's metadata cache, which triggers once the file has been indexed. At that point, it automatically runs the `Excalidraw: Toggle between Excalidraw and Markdown mode` command from the Command Palette and removes the event listener.

```js
const path = tp.file.path(true);
const handler = (file) => {
  if(file.path === path) {
    app.metadataCache.off("changed", handler);
    app.commands.executeCommandById("obsidian-excalidraw-plugin:toggle-excalidraw-view")
  }
}
app.metadataCache.on("changed", handler);
```