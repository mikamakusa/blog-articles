+++
date = "2017-01-24T15:29:49+02:00"
draft = "false"
title = "Le Labo #21 | A la chasse aux démons"

+++

# Avant de commencer
Il est vrai que le titre peut laisser songeur quant à mon intention.  
Non, il n'y a rien de religieux....Mon blog n'a pour sujet que l'informatique.  
Dans ce billet, je vais aborder la démonisation (ou daemonization) d'application à l'aide de Salt via des states et des pillars.
Si vous recherchez des formules relatives à l'installation de ces applications, vous en trouverez plein sur internet.

# Ce qu'il nous faut
Dans ce labo, je vais avoir besoin de plusieurs serveurs : 

- Un serveur Salt  
- Deux serveurs sur lequel seront installés **Logstash** et **Elasticsearch** ainsi que **Salt-Minion**  
- Les serveurs sur lesquels seront installés **logstash** et **Elasticsearch** seront équipés de **CentOs 6** et **CentOs 7**, j'expliquerais pourquoi plus tard.


# Comment installer Elasticsearch et Logstash ?
Bon, je ne vais rien vous apprendre, mais les packages sont disponible sur le site officiel...ceci dit, je ne vais pas utiliser les *.rpm* ou les *.deb* mais les archives *.tar.gz*, ce qui me permettra de gérer de bout en bout le processus d'installation et de daemonization.  

Donc, une fois ceux-ci téléchargés, on les décompresse dans un dossier bien spécifique :  
`tar xzf /chemin/vers/archive -C /chemin/vers/dossier`

## Sur CentOS 6
Similaire à **RedHat 6** dans la gestion des services/daemons, les services se trouvent dans le dossier */etc/init.d/* et sont, en fait, de simples executables comme celui qui suit : 

Exemple pour ELasticSearch: 

	#!/bin/bash
	export JAVA_HOME=/opt/java
	RETVAL=0
	ARGS="@0"
	prog="elasticsearch"
	HOME="/opt/elasticsearch"
	CONFIG="/opt/elasticsearch/conf/elasticsearch.yml"
	start () {
		echo -n $"Starting $prog"
		if [ -r "$CONFIG" ]
		then
			/opt/elasticsearch/bin/elasticsearch > ${HOME}/logs/elasticsearch.log 2>&1 &
			RETVAL=$?
			PID=$!
			echo
			[ $RETVAL -eq 0 ] && echo $PID > ${HOME}/run/elasticsearch.pid
	}
	stop () {
		echo -, $"Stopping $prog"
		kill 'cat ${HOME}/run/elasticsearch.pid'
		RETVAL=$?
		echo
		[ $RETVAL -eq 0 ] && rm -f ${HOME}/run/elasticsearch.pid
	}
	status () {
		if [ -f ${HOME}/run/elasticsearch.pid]
		then
			echo "$prog is running with PID : " cat ${HOME}/run/elasticsearch.pid
		else
			echo "$prog is not running"
		fi
	}
	case "$1" in
		start)
			start
			;;
		stop)
			stop
			;;
		restart|reload)
			stop
			start
			;;
		*)
			echo $"Usage: $0 {start|stop|status|restart|reload}"
			exit 1
	esac
	exit $?

## Sur CentOS 7
DE la même manière que **CentOS 6** est similaire à **RedHat 6** dans la gestion de services, **CentOs 7** fait de même que **RedHat 7**. Les services se trouvent dans **/lib/systemd/system** et ressemblent à ça : 

Exemple pour Logstash

	[Unit]
	Description=Logstash
	After=network.target
	[Service]
	Environment=LOG_DIR=/opt/logstash/logs
	Environment=CONF_DIR=/opt/logstash/conf
	Environment=BIN_DIR=/opt/logstash/bin
	Environment=PID_DIR=/opt/logstash/run
	Environment=JAVA_HOME=/opt/java
	User=root
	Group=root
	ExecStart=
	ExecStart=/opt/logstash/bin agent -config ${CONF_DIR}/logstaash.conf
	ExecReload=/bin/kill -s HUP ${MAINPID}
	ExecStop=/bin/kill -WINCH ${MAINPID}
	StandardOutput=journal
	StandardError=inherit
	LimitBOFILE=65535
	TimeoutStopSec=0
	KillSignal=SIGTERM
	SendSIGKILL=no
	SuccessExitStatus=143
	[Install]
	WantedBy=multi-user.target

## Via SaltStack
En fait, l'automatisation de la création des démons via Salt sera relativement simple à réaliser, mais j'ai voulu faire quelque chose d'universel qui puisse aussi bien servir pour **Logstash** que pour **Elasticsearch**.  
J'ai hésité un petit moment entre injecter du jinja, des grains et des pillars...mais il s'avère que les deux derniers sont plus efficace pour mon usage.

### Le State

	{% set logs = conf.directories.logs %}
	{% set daemon = pillar['get'](daemon) %}
	{% set osMajorRelease = grains['get'('osmajorrelease')] %}
	{% for applicatio, conf in daemon.instance.iteritems() %}
		{% set prefix = conf.prefix %}
		{% set logs = conf.directories.logs %}
		{% set confs = conf.directories.conf %}
		{% set pid = conf.directories.pid %}
		{% set java = conf.directories.jvm %}
		{% set user = conf.user %}
		{% set group = conf.group %}
		{% set bin = conf.bin %}
		{% set script = conf.script %}
		{% set install_dir = conf.install_dir %}
		{% set command = conf.command %}
		{% if osMajorRelease == '7' %}
			{{ application }}_systemd_unit:
				- name: /lib/systemd/system/{{ application }}.service
				- contents:
					- '[Unit]'
					- Description={{ application }}
					- After=network.target
					- '[Service]'
					- Environment=LOG_DIR={{ prefix }}/{{ application }}/{{ logs }}
					- Environment=CONF_DIR={{ prefix }}/{{ application }}/{{ confs }}
					- Environment=BIN_DIR={{ prefix }}/{{ bin }}
					- Environment=PID_DIR={{ prefix }}/{{ application }}/{{ pid }}
					- Environment=JAVA_HOME={{ java.dir }}{{ java.vers }}
					- User={{ user }}
					- Group={{ group }}
					- ExecStart=
					- ExecStart={{ install_dir }}/{{ bin }}/{{ command }}
					- ExecReload=/bin/kill -s HUP ${MAINPID}
					- ExecStop=/bin/kill -WINCH ${MAINPID}
					- StandardOutput=journal
					- StandardError=inherit
					- LimitBOFILE=65535
					- TimeoutStopSec=0
					- KillSignal=SIGTERM
					- SendSIGKILL=no
					- SuccessExitStatus=143
					- '[Install]'
					- WantedBy=multi-user.target
			{{ application }}_system_enable:
				cmd.run: 
					- name: if [[ $(systemctl list-unit-files | grep {{ application }} | awk '{print $2}') != 'enabled' ]]; then systemctl enable {{ application }}; fi
			{{ application }}_system_reload:
				module.run:
					- name: service.systemctl_reload
			{{ application }}_running:
				cmd.run:
					- name: systemctl restart {{ application }}
		{% else %}
			{{ application }}_service_unit:
				file.symlink:
					- name: /etc/init.d/{{ application }}
					- target: {{ prefix }}{{ bin }}{{ script }}
			{{ application }}_service_enable:
				cmd.run:
				- name: if [[ $(chkconfig --list | grep {{ application }} | awk '{print $4}') != "2:on" ]]; then chkconfig {{ application }} on; fi
			{{ application }}_service_running:
				cmd.run:
					name: service {{ application }} start
		{% endif %}
	{% endfor %}

### Le pillar

	daemon:
		instances:
			logstash:
				user: root
				group: root
				install_dir: /opt
				prefix: /opt
				bin: /logstash/bin/
				script: logstash.sh
				command: logstash agent --config ${CONF_DIR}/logstash.conf
				directories:
					logs: /logs
					confs: /conf
					pids: /run
					jvm:
						dir: /opt/jdk
						vers: 8
			elasticsearch:
				user: root
				group: root
				install_dir: /opt
				prefix: /opt
				bin: /elasticsearch/bin/
				script: elasticsearch
				command: elasticsearch --pidfile ${PID_DIR}/elasticsearch.pid
				directories:
					logs: /logs
					confs: /conf
					pids: /run
					jvm:
						dir: /opt/jdk
						vers: 8

# Pour finir
Avec tout cela, vous avez un petit avant-goût de ce qui est faisable avec Salt.
Certes, je me suis limité à quelque chose de relativement simple, le tout est de savoir comment organiser les données pour, par la suite, les sortir et les pillariser avec **Salt**.