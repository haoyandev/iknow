# cra项目引入Less

1. 安装 react-app-rewired customize-cra

   ```js
   yarn add customize-cra react-app-rewired --dev
   ```

2. 目根目录创建文件config-overrides.js

   ```js
   const { override,addLessLoader} = require("customize-cra");
   module.exports = override(
   　　　addLessLoader({
           javascriptEnabled: true,
           modifyVars: {}
       }),
   );
   ```

3. 修改package.json

   ```js
   "scripts": {
       "start": "react-app-rewired start",
       "build": "react-app-rewired build",
       "test": "react-app-rewired test",
       "eject": "react-app-rewired eject"
     },
   ```

   

