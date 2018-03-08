麒麟麻将 creator1.8.0 部署遇到的问题总结 

1. IP 地址 
	server:	configs_mac.js
			var HALL_IP = "192.168.0.71";//如果非本机访问，这里要变	
			PSWD:'root',//如果连接失败，请检查这里
			DB:'majiang',//如果连接失败，请检查这里

	client:	HTTP.js
			var URL = "http://192.168.0.71:9000";
 2. 启动命令
	注意 第二个参数的路径(../configs_mac.js)
	node account_server/app.js ../configs_mac.js
 	node hall_server/app.js ../configs_mac.js
	node game_server/app.js ../configs_mac.js
3.	ImageLoader.js 显示头像图片
	
	url = ret.headimgurl;
	加载图片时指定格式
	cc.loader.load({url:url,type:"jpg"},function (err,tex) {})
4.	loadingLogic.js 加载 textures 方法用 loadRes
	        cc.loader.loadRes("textures", function (err, assets) {})

5.	jsb_socketio.cpp 和服务器版本要匹配
	
	static bool SocketIO_emit(se::State& s)
{
    const auto& args = s.args();
    int argc = (int)args.size();
    SIOClient* cobj = (SIOClient*)s.nativeThisObject();

    if (argc >= 1)
    {
        bool ok = false;
        std::string eventName;
        ok = seval_to_std_string(args[0], &eventName);
        SE_PRECONDITION2(ok, false, "Converting eventName failed!");
        
        std::string payload;
        if (argc >= 2)
        {
            const auto& arg1 = args[1];
            // Add this check to make it compatible with old version.
            // jsval_to_std_string in v1.6 returns empty string if arg1 is null or undefined
            // while seval_to_std_string since 1.7.2 follows JS standard to return "null" or "undefined".
            // Therefore, we need a workaround to make it be compatible with versions lower than v1.7.
            if (!arg1.isNullOrUndefined())
            {
                ok = seval_to_std_string(arg1, &payload);
                SE_PRECONDITION2(ok, false, "Converting payload failed!");
            }
        }
        cobj->emit(eventName, payload);
        return true;
    }

    SE_REPORT_ERROR("Wrong number of arguments: %d, expected: %d", argc, 2);
    return false;
}

6.	微信登陆
	
￼
	需要更改 appControll.mm  和 添加 info.plist微信启动游戏回调参数
	
7. 项目设置 需要勾选collider 模块
	
￼

8. 	调试服务器 .vscode/launch.json
	        {
            "name": "启动",
            "type": "node",
            "request": "launch",
            // "program": "${workspaceRoot}/majiang_server/majiang_server.js",
            "program": "${workspaceRoot}/hall_server/app.js",
            "stopOnEntry": false,
            "args": ["${workspaceRoot}/configs_mac.js"],
            "cwd": "${workspaceRoot}",
            "preLaunchTask": null,
            "runtimeExecutable": null,
            "runtimeArgs": [
                "--nolazy"
            ],
            "env": {
                "NODE_ENV": "development"
            },
            "externalConsole": false,
            "sourceMaps": false,
            "outDir": null
        },


10. 语音 SDK 不支持 ios64, 已更新在 native 下
