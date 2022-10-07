# VSCode

## Run NodeJS in a container

#### Run / Debug

Once F5 / Ctrl+F5 pressed IDE will look at `.vscode/launch.json`&#x20;

```
{
    "configurations": [
        {
            "name": "Docker Node.js Launch",
            "type": "node",
            "request": "launch",
            "platform": "node",
            "program": "${workspaceFolder}/src/index.js"
        },
    ]
}
```

Line (8) will be used to run the script

#### Run task

Ctrl + P -> type "task " (space at the end). There will menu appear with selection of available options to run. Those options are take from `.vscode/tasks.json`&#x20;

```
{
	"version": "2.0.0",
	"tasks": [
		{
			"type": "npm",
			"script": "start",
			"problemMatcher": [],
			"label": "npm: start",
			"detail": "npm run build:live"
		},
		{
			"type": "npm",
			"script": "run",
			"problemMatcher": [],
			"label": "npm: run",
			"detail": "node src/index.js"
		}
		]
}
```

Once task selected line (9) or (16) will be used as marching order

#### One more place to check

package.json

```
{
  "name": "eslint-starter",
  "version": "1.0.0",
  "description": "Tiny eslint example",
  "main": "src/server.js",
  "type": "module",
  "scripts": {
    "run": "node src/index.js",
    "start": "node .",
    "lint": "eslint src",
    "test": "npm run lint && echo \"Write some tests!\"",
    "watch": "nodemon --watch src --exec \"npm run test && npm run start\""
  },
  "author": "Michael Trotter",
  "license": "MIT",
  "devDependencies": {
    "nodemon": "^1.8.1"
  },
  "dependencies": {
    "@tensorflow/tfjs-core": "^3.13.0",
    "@tensorflow/tfjs-node": "^3.13.0",
    "glob": "^7.1.6",
    "make-dir": "^3.1.0",
    "minipass": "^3.1.6"
  }
}
```

Line (5) will run by other scripts once they do `require("this module")`.

Lines (8) - (12) used on different lifecycle events ??????? (not clear yet)
