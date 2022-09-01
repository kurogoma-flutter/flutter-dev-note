### Flutter開発用のsetting.json
#### .vscode/setting.json に格納する

```
{
  "debug.openDebug": "openOnDebugBreak",
  "editor.formatOnSave": true,
  "editor.formatOnType": true,
  "editor.rulers": [80],
  "editor.renderWhitespace": "all",
  "editor.renderControlCharacters": true,
  "editor.minimap.enabled": false,
  "editor.bracketPairColorization.enabled": true,
  "[dart]": {
    "editor.selectionHighlight": false,
    "editor.suggest.snippetsPreventQuickSuggestions": false,
    "editor.suggestSelection": "recentlyUsedByPrefix",
    "editor.tabCompletion": "onlySnippets",
    "editor.wordBasedSuggestions": false,
    "editor.codeActionsOnSave": [
      "source.organizeImports",
      "source.addMissingImports",
      "source.fixAll"
    ]
  },
  "dart.flutterSdkPath": ".fvm/flutter_sdk",
  "dart.debugSdkLibraries": false,
  "dart.showSkippedTests": false,
  "dart.runPubGetOnPubspecChanges": true,
  "search.exclude": {
    "**/.fvm": true,
    "**/*.freezed.dart": true,
    "**/*.g.dart": true
  },
  "files.watcherExclude": {
    "**/.fvm": true
  },
  "git.autofetch": true,
  "explorer.confirmDragAndDrop": false,
  "explorer.fileNesting.enabled": true,
  "explorer.fileNesting.expand": false,
  "explorer.fileNesting.patterns": {
    "pubspec.yaml": ".flutter-plugins, .packages, .dart_tool, .flutter-plugins-dependencies, .metadata, template.iml, .packages, pubspec.lock, build.yaml, analysis_options.yaml, all_lint_rules.yaml, flutter_launcher_icons-*.yaml, flutter_native_splash.yaml, lefthook.yaml, Makefile",
    "firebase.json": ".firebaserc, firestore.rules, firestore.indexes.json,storage.rules, remoteconfig.template.json",
    ".env.example": ".env.*",
    ".gitignore": ".gitattributes, .gitmodules, .gitmessage, .mailmap, .git-blame*",
    "readme.*": "authors, backers.md, changelog*, citation*, code_of_conduct.md, codeowners, contributing.md, contributors, copying, credits, governance.md, history.md, license*, maintainers, readme*, security.md, sponsors.md",
    "*.dart": "$(capture).g.dart, $(capture).freezed.dart"
  },
  "cSpell.words": ["linkify", "rebord"]
}
```