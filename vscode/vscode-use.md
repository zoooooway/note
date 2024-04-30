# vscode相关使用记录

## vue

### eslint, vetur
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

### 无法跳转到Component
引入Component时, 不要忽略后缀名`.vue`.
