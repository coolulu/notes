#1.杀死占用的端口进程 或者 遇到 Address already in use 

 fuser -k 9999/tcp

#2.生成随机密钥：
cat /dev/urandom | base64 | head -n1 

#3.统计工程树下的所有 .cpp .h源码的非空行行数：
find . -name '*.cpp' | xargs cat | grep . | wc -l
find . -name '*.h' | xargs cat | grep . | wc -l


