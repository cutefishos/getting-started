# Adding a Language

Cutefish is written in US English. The project maintains ten catalogs
itself: `es`, `pt_BR`, `fr`, `de`, `ru`, `ja`, `ko`, `it`, `zh_CN` and
`zh_TW`. A wrong translation in one of them is corrected by a pull request
against `po/<code>.po`; later updates keep the correction. Any other
language comes in as a pull request, one per repository, and is welcome
whether a person or an agent wrote it. The text lives in gettext catalogs:

| Repository | Template |
|---|---|
| cutefish-toolkit | `po/cutefish-toolkit.pot` |
| cutefish-framework | `po/cutefish-framework.pot` |
| cutefish-core | `po/cutefish-core.pot` |
| cutefish-compositor | `po/cutefish-compositor.pot` |
| cutefish-dm | `po/cutefish-dm.pot` |
| filemanager | `po/cutefish-filemanager.pot` |
| settings | `po/cutefish-settings.pot` |
| terminal | `po/cutefish-terminal.pot` |
| text-editor | `po/cutefish-text-editor.pot` |
| video-player | `po/cutefish-video-player.pot` |
| calendar | `po/cutefish-calendar.pot` |
| system-monitor | `po/cutefish-system-monitor.pot` |
| calculator | `po/cutefish-calculator.pot` |
| preview | `po/cutefish-preview.pot` |

A language is `po/<code>.po` next to the template: `ja.po`, `de.po`, or
with a territory where it matters, `zh_CN.po`, `zh_TW.po`, `pt_BR.po`. The
catalogs already in `po/` show the finished form.

1. Make the catalog from the template, in each repository:

   ```shell
   msginit --no-translator --no-wrap -l ja_JP.UTF-8 \
       -i po/cutefish-system-monitor.pot -o po/ja.po
   ```

   Check its `Plural-Forms` header: msginit leaves a placeholder for some
   languages, Chinese among them; those without plural forms take
   `nplurals=1; plural=0;`.

2. Translate every `msgstr`:
   - Keep each `{name}` placeholder exactly as it is written. Their order
     may change to suit the sentence.
   - Give every plural form (`msgstr[0]`, `msgstr[1]`, ...) for languages
     that have them; `{n}` is the count.
   - `msgctxt` tells two uses of the same English apart, and
     `#. TRANSLATORS:` comments explain short or ambiguous text; neither is
     shown.
   - Desktop `Keywords` stay a list of words, each followed by `;`.
   - Leave product names (Cutefish) and technical names (Wi-Fi, Bluetooth,
     VPN, PID, MiB) as they are unless the language has its own form.
   - Leave `Last-Translator` and `Language-Team` empty; the commit names the
     translator.

3. Check the catalogs and see them in use:

   ```shell
   cmake --build build/<repository> --target check-translations
   sudo cmake --install build/<repository>
   ```

   check-translations runs `msgfmt --check`, which also rejects a missing or
   misspelt placeholder. Installed catalogs go to
   `/usr/share/locale/<code>/LC_MESSAGES/`. Pick the language in Settings >
   Language and drag it to the top: open windows switch to it at once.

When the English changes, maintainers update the templates with
`cmake --build build/<repository> --target update-translations`, which also
merges every catalog; entries whose English changed are marked fuzzy and
show in English until they are translated again. Do not edit the `.pot`
files by hand.
