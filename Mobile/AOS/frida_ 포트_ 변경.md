
[adb shell]
./frida-server -l localhost:8088 
[cmd]
adb forward tcp:8088 tcp:8088 
[frida script 실행]
frida -H localhost:8088 -f com.sampleapp -l test.js