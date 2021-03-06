# webpack-plugin-chokidar

let webpack monitor changes in files that are not in the dependency graph.

## Docs

[English](README.md) | [中文](README_zh.md)

## Foreword

Those who have used the [umi framework](https://umijs.org/docs/routing) must be impressed with its conventional routing function. You only need to create a new file in the page folder without any configuration, and you can jump directly to this page. The implementation principle is actually very simple. You only need to monitor the addition and deletion of files in the page folder, and run the script to update the routing table files.

But as we all know, Webpack can only monitor the changes of files in the file dependency graph. When a file that is not in the dependency graph is created, modified, or deleted, there is no way to monitor it.

`webpack-plugin-chokidar` is a tool that allows Webpack to monitor changes in files that are not in the dependency graph. In the callback function, you can execute your automation script. It integrates [chokidar](https://github.com/paulmillr/chokidar), redesigns its API, and only needs to pass in a configuration when using it.

## Install

```bash
npm i webpack-plugin-chokidar -D
```

## Config

| Item    | Type                          | Mapping                                                        | Meaning                           |
| ------- | ----------------------------- | -------------------------------------------------------------- | --------------------------------- |
| file    | string                        | chokidar.watch(**file**, opt)                                  | The file or dir you want to watch |
| opt     | [WatchOptions](src/types.ts)  | chokidar.watch(file, **opt**)                                  | watch options                     |
| actions | [ChokidarEvent](src/types.ts) | watcher['on' \| 'close' \| 'add' \| 'unwatch' \| 'getWatched'] | watch callback                    |

You can check the [type file](src/types.ts) and [chokidar document](https://github.com/paulmillr/chokidar#api) to see more detailed configuration

## Example

This is a simple example which can upload file to remote computer when monitor target file is changed.

```js
const shell = require('shelljs')
const debounce = require('lodash.debounce')
const path = require('path')

const uploadFileToRemote = debounce(() => {
  const dirPath = path.resolve(__dirname, '../dist')
  shell.exec(
    `sshpass -p "test12345" scp -r ${dirPath} acm@192.168.1.1:/Users/acm/Desktop/spider`
  )
}, 1000)

// ...some webpack code

new WebPackPluginChokidar({
  chokidarConfigList: [
    {
      file: '../dist/*',
      opt: { persistent: true, ignoreInitial: true },
      actions: {
        on: {
          change: ({ compiler, compilation, watcher }, path) => {
            uploadFileToRemote()
          },
        },
      },
    },
  ],
})
```
