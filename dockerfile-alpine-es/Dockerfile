# [FROM]（指定基础image）
# 构建指令，必须指定且需要在Dockerfile其他指令的前面。后续的指令都依赖于该指令指定的image。FROM指令指定的基础image可以是官方远程仓库中的，也可以位于本地仓库。
# FROM <image> 指定基础image为该image的最后修改的版本。或者： FROM <image>:<tag>  指定基础image为该image的一个tag版本
FROM openjdk:8-jre-alpine
# [MAINTAINER]（用来指定镜像创建者信息）
#构建指令，用于将image的制作者相关的信息写入到image中。当我们对该image执行docker inspect命令时，输出中有相应的字段记录该信息。
MAINTAINER psiitoy "psiitoy@126.com"

# [RUN] 构建指令，RUN可以运行任何被基础image支持的命令。如基础image选择了ubuntu，那么软件管理部分只能使用ubuntu的命令。
# ensure elasticsearch user exists 创建用户组 用户 es启动不可以用root账户
RUN addgroup -S elasticsearch && adduser -S -G elasticsearch elasticsearch

RUN apk update
# grab su-exec for easy step-down from root
# and bash for "bin/elasticsearch" among others
RUN apk add --no-cache 'su-exec>=0.2' bash

# [WORKDIR]（切换目录）设置指令，可以多次切换(相当于cd命令)，对RUN,CMD,ENTRYPOINT生效。
WORKDIR /usr/share/elasticsearch
# [ENV]（用于设置环境变量）  构建指令，在image中设置一个环境变量。
ENV PATH /usr/share/elasticsearch/bin:$PATH
#ENV ELASTICSEARCH_VERSION 2.4.5
ENV ELASTICSEARCH_VERSION 2.1.2
#ENV ELASTICSEARCH_VERSION 5.4.2

ENV ES_PKG_NAME elasticsearch-$ELASTICSEARCH_VERSION
ADD $ES_PKG_NAME/ /usr/share/elasticsearch

RUN set -ex; \
	\
	# 安装常用组件
	apk add --no-cache --virtual \
		ca-certificates \
		gnupg \
		openssl \
		tar \
		openssh \
		vim \
		curl \
	; \
	\
	# mkdir -p 递归建立根目录
	mkdir -p ./plugins; \
	# shell循环遍历语法 for xxx in; do echo $xxx done; 循环赋权限
	for path in \
		./data \
		./logs \
		./config \
		./config/scripts \
	; do \
		mkdir -p "$path"; \
		chown -R elasticsearch:elasticsearch "$path"; \
	done; \
	\
# we shouldn't need much RAM to test --version (default is 2gb, which gets Jenkins in trouble sometimes)
# 设置JVM内存
	export ES_JAVA_OPTS='-Xms256m -Xmx256m'; \
# 打印版本号(1.x 跟 1.x以上 语法不同而已)
	if [ "${ELASTICSEARCH_VERSION%%.*}" -gt 1 ]; then \
		elasticsearch --version; \
	else \
# elasticsearch 1.x doesn't support --version
# but in 5.x, "-v" is verbose (and "-V" is --version)
		elasticsearch -v; \
#    plugin install mobz/elasticsearch-head; \
	fi

# 复制config文件夹到容器
COPY config ./config

# [VOLUME]（指定挂载点)）设置指令，使容器中的一个目录具有持久化存储数据的功能，该目录可以被容器本身使用，也可以共享给其他容器使用。
# 我们知道容器使用的是AUFS，这种文件系统不能持久化数据，当容器关闭后，所有的更改都会丢失。当容器中的应用有持久化数据的需求时可以在Dockerfile中使用该指令。
# 加上VOLUME 这样es 的 data文件夹下的内容可以持久化了
VOLUME /usr/share/elasticsearch/data
VOLUME /usr/share/elasticsearch/logs

COPY docker-entrypoint.sh /

# 容器需要开放ES的9200和9300端口
EXPOSE 9200 9300 8300
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["elasticsearch"]