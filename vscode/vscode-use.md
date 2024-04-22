# vscode相关使用记录

## workspace

### vue, eslint, vetur
```json
{
	"folders": [
		{
			"path": "."
		}
	],
	"settings": {
		"commentTranslate.hover.concise": true,
		"eslint.enable": true,
		"eslint.format.enable": true,
		"editor.codeActionsOnSave": {
			"source.fixAll.eslint": "explicit"
		},
		"eslint.workingDirectories": [
			"."
		],
		"vetur.experimental.templateInterpolationService": true,
		"vetur.validation.interpolation": false
	}
}
```