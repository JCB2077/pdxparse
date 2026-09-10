# pdxparse wiki template pack

Wikitext for the templates that pdxparse output transcludes, plus notes on the
places where these four wikis conflict with what the parser assumes.

Everything here is plain wikitext. Create the page named in each file's heading
and paste the contents in; nothing needs to be installed on the wiki server.

## What pdxparse assumes exists

| Template | Emitted by | Call shape |
| --- | --- | --- |
| `{{Version}}` | every generated file | `{{Version|1.16}}` |
| `{{buffer}}` | table headers | `{| class="mildtable" {{buffer}}` |
| `{{iconbox}}` / `{{Focusbox}}` | national focuses, ideas | `{{Focusbox|image=X.png\|Name\|Desc}}` |
| `{{icon}}` | ~280 call sites across the parser | `{{icon|time}}` |
| `{{effectbox}}` | idea and spirit effect boxes | `{{effectbox|Name|file=X.png\|Desc\|modifiers=…\|indent=N}}` |
| `{{hover}}` | long trigger text | `{{hover|shown|tooltip}}` |
| `{{color}}`, `{{green}}`, `{{red}}` | coloured numbers | `{{green|+10}}` |
| `{{path}}` | generated-template footers | `{{path|common/ideas/x.txt}}` |
| `mildtable` (CSS class, not a template) | every table | `MediaWiki:Common.css` |

## Per-wiki status

Checked September 2026. Re-check before relying on it; wikis change.

| | Old World Blues | Equestria at War | The New Order | Red Flood |
| --- | --- | --- | --- | --- |
| `Icon` | compatible | compatible | **conflicts** | not checked |
| `Iconbox` | **conflicts** | **conflicts** | **absent** | not checked |
| `Buffer` | present | not checked | **absent** | not checked |
| `Effectbox` | present | not checked | **absent** | not checked |

**`Iconbox` conflicts.** On Old World Blues and Equestria at War, `Iconbox` is an
achievements template: it takes `name`, `description` and an optional `link`, and
derives the icon from the *name*. pdxparse passes the icon as a named `image=`
parameter, which that template ignores — so the positional name and description
land correctly but the icon resolves to `File:<focus name>.png` and comes out
broken. Installing the Paradox-wiki `Iconbox` over the top would break every
existing achievement list, so the fix is a separate template: install
`Template-Focusbox.wiki` as `Template:Focusbox` and set

```yaml
focus_box_template: "Focusbox"
```

in `settings.yml`. The setting defaults to `iconbox`, so paradoxwikis output is
unchanged if you leave it alone.

**`Icon` conflicts on The New Order.** TNO's `Template:Icon` is
`[[File:{{{iconfile}}}|22px]]` — a named parameter, not a positional icon code.
Every `{{icon|time}}` in pdxparse output expands to `[[File:|22px]]`. Do not
replace the template; existing pages depend on it. `Template-Icon-compat-patch.wiki`
shows how to make it accept both forms, which leaves current pages working.

## Files

| File | Install as |
| --- | --- |
| `Template-Focusbox.wiki` | `Template:Focusbox` |
| `Template-Buffer.wiki` | `Template:Buffer` |
| `Template-Version.wiki` | `Template:Version` |
| `Template-Hover.wiki` | `Template:Hover` |
| `Template-Color.wiki` | `Template:Color` |
| `Template-Green.wiki` | `Template:Green` |
| `Template-Red.wiki` | `Template:Red` |
| `Template-Path.wiki` | `Template:Path` |
| `Template-Icon-compat-patch.wiki` | not a template — instructions for patching an existing `Template:Icon` |
| `Common.css` | append to `MediaWiki:Common.css` |

## Notes on the templates

They avoid Scribunto, so they work on a wiki without the Lua extension. Where a
parameter can be absent they use `{{#if:}}` and render nothing rather than
leaving an empty cell or a broken file link — pdxparse omits parameters for data
the game script did not supply, and that happens often in mods.

`Template:Buffer` must have no newline inside `<includeonly>`, because it is
transcluded inside a table's opening line.
