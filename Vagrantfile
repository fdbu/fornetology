#
# For use all Virtual Machines required 4.5 GB RAM
#

Vagrant.configure("2") do |config|

  config.ssh.insert_key = false

# ---------------- webserver ----------------------------------------------------------------

  config.vm.define "webserver" do |webserver|
                webserver.vm.hostname = "webserver"
                webserver.vm.box = "ubuntu/bionic64"
                webserver.vm.network "private_network", ip: "172.16.0.10"

        webserver.vm.provider "virtualbox" do |vb|
                vb.name = "webserver"
                vb.cpus = 1
                vb.memory = "512"
        end

        webserver.vm.provision "shell", inline: <<-SHELL
                apt-get update
                apt-get install htop nginx -y
		echo "events {"                                                                  > /etc/nginx/nginx.conf
		echo "	worker_connections 1024;"                                                >> /etc/nginx/nginx.conf
		echo "}"                                                                         >> /etc/nginx/nginx.conf
		echo ""                                                                          >> /etc/nginx/nginx.conf
		echo "http {"                                                                    >> /etc/nginx/nginx.conf
		echo "	include /etc/nginx/mime.types;"                                          >> /etc/nginx/nginx.conf
		echo "	include /etc/nginx/conf.d/*.conf;"                                       >> /etc/nginx/nginx.conf
		echo "	include /etc/nginx/sites-enabled/default;"                               >> /etc/nginx/nginx.conf
		echo "	default_type application/octet-stream;"                                  >> /etc/nginx/nginx.conf
		echo ""                                                                          >> /etc/nginx/nginx.conf
		echo "	log_format json escape=json"                                             >> /etc/nginx/nginx.conf
		echo "		'{'"                                                             >> /etc/nginx/nginx.conf
		echo "			'\\"Authorization\\":\\"\\$http_authorization\\",'"              >> /etc/nginx/nginx.conf
		echo "			'\\"RequestTime\\":\\"\\$time_iso8601\\",'"                      >> /etc/nginx/nginx.conf
		echo "			'\\"RemoteAddress\\":\\"\\$remote_addr\\",'"                     >> /etc/nginx/nginx.conf
		echo "			'\\"RemotePort\\":\\"\\$remote_port\\",'"                        >> /etc/nginx/nginx.conf
		echo "			'\\"RemoteUser\\":\\"\\$remote_user\\",'"                        >> /etc/nginx/nginx.conf
		echo "			'\\"RequestHost\\":\\"\\$host\\",'"                              >> /etc/nginx/nginx.conf
		echo "			'\\"RequestPort\\":\\"\\$server_port\\",'"                       >> /etc/nginx/nginx.conf
		echo "			'\\"RequestMethod\\":\\"\\$request_method\\",'"                  >> /etc/nginx/nginx.conf
		echo "			'\\"RequestPath\\":\\"\\$request_uri\\",'"                       >> /etc/nginx/nginx.conf
		echo "			'\\"RequestBody\\":\\"\\$request_body\\",'"                      >> /etc/nginx/nginx.conf
		echo "			'\\"ResponseStatus\\":\\"\\$status\\",'"                         >> /etc/nginx/nginx.conf
		echo "			'\\"Upstream\\":\\"\\$upstream_addr\\",'"                        >> /etc/nginx/nginx.conf
		echo "			'\\"UpstreamPath\\":\\"\\$uri\\",'"                              >> /etc/nginx/nginx.conf
		echo "			'\\"UpstreamResponseTime\\":\\"\\$upstream_response_time\\"'"    >> /etc/nginx/nginx.conf
		echo "		'}';"                                                            >> /etc/nginx/nginx.conf
		echo ""                                                                          >> /etc/nginx/nginx.conf
		echo "	access_log syslog:server=172.16.0.150:5555 json;"                        >> /etc/nginx/nginx.conf
		echo "}"                                                                         >> /etc/nginx/nginx.conf
		systemctl restart nginx
		systemctl enable nginx
        SHELL
  end

# ---------------- elk -------------------------------------------------------------------

  config.vm.define "elk" do |elk|
                elk.vm.hostname = "elk"
                elk.vm.box = "ubuntu/bionic64"
                elk.vm.network "private_network", ip: "172.16.0.150"
		elk.vm.network "public_network", bridge: "wlp2s0", ip: "192.168.168.150"

        elk.vm.provider "virtualbox" do |vb|
                vb.name = "elk"
                vb.cpus = 2
                vb.memory = "4096"
        end

        elk.vm.provision "shell", inline: <<-SHELL
		wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
		echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee -a /etc/apt/sources.list.d/elastic-7.x.list
                apt-get update
                apt-get install htop apt-transport-https elasticsearch logstash kibana -y
                echo "discovery.type: single-node" >> /etc/elasticsearch/elasticsearch.yml
                echo "network.host: '_local_'"     >> /etc/elasticsearch/elasticsearch.yml
                systemctl enable elasticsearch
                systemctl restart elasticsearch
		echo "input {"                                                                                                                           >> /etc/logstash/conf.d/logstash.conf
		echo "  syslog {"                                                                                                                        >> /etc/logstash/conf.d/logstash.conf
		echo "    port => 5555"                                                                                                                  >> /etc/logstash/conf.d/logstash.conf
		echo '    tags => "nginx"'                                                                                                               >> /etc/logstash/conf.d/logstash.conf
		echo "  }"                                                                                                                               >> /etc/logstash/conf.d/logstash.conf
		echo "}"                                                                                                                                 >> /etc/logstash/conf.d/logstash.conf
		echo ""                                                                                                                                  >> /etc/logstash/conf.d/logstash.conf
		echo "filter{"                                                                                                                           >> /etc/logstash/conf.d/logstash.conf
		echo ""                                                                                                                                  >> /etc/logstash/conf.d/logstash.conf
		echo "    json{"                                                                                                                         >> /etc/logstash/conf.d/logstash.conf
		echo '        source => "message"'                                                                                                       >> /etc/logstash/conf.d/logstash.conf
		echo "    }"                                                                                                                             >> /etc/logstash/conf.d/logstash.conf
		echo ""                                                                                                                                  >> /etc/logstash/conf.d/logstash.conf
		echo "    date {"                                                                                                                        >> /etc/logstash/conf.d/logstash.conf
		echo '       match  => ["RequestTime","ISO8601"]'                                                                                        >> /etc/logstash/conf.d/logstash.conf
		echo "    }"                                                                                                                             >> /etc/logstash/conf.d/logstash.conf
		echo ""                                                                                                                                  >> /etc/logstash/conf.d/logstash.conf
		echo "    mutate {"                                                                                                                      >> /etc/logstash/conf.d/logstash.conf
		echo '        remove_field => ["message","timestamp","RequestTime","facility","facility_label","severity","severity_label","priority"]'  >> /etc/logstash/conf.d/logstash.conf
		echo "    }"                                                                                                                             >> /etc/logstash/conf.d/logstash.conf
		echo ""                                                                                                                                  >> /etc/logstash/conf.d/logstash.conf
		echo "}"                                                                                                                                 >> /etc/logstash/conf.d/logstash.conf
		echo ""                                                                                                                                  >> /etc/logstash/conf.d/logstash.conf
		echo "output {"                                                                                                                          >> /etc/logstash/conf.d/logstash.conf
                echo 'if [program] == "nginx" {'                                                                                                         >> /etc/logstash/conf.d/logstash.conf
                echo "    elasticsearch {"                                                                                                               >> /etc/logstash/conf.d/logstash.conf
                echo '      hosts => ["http://localhost:9200"]'                                                                                          >> /etc/logstash/conf.d/logstash.conf
                echo '      index => "nginx-index"'                                                                                                      >> /etc/logstash/conf.d/logstash.conf
                echo "    }"                                                                                                                             >> /etc/logstash/conf.d/logstash.conf
                echo "  }"                                                                                                                               >> /etc/logstash/conf.d/logstash.conf
		echo "}"                                                                                                                                 >> /etc/logstash/conf.d/logstash.conf
		systemctl enable logstash
		systemctl start logstash
                echo "elasticsearch.hosts: ["http://localhost:9200"]"   >> /etc/kibana/kibana.yml
                echo "server.host: "0.0.0.0""                           >> /etc/kibana/kibana.yml
                echo "server.port: "8080""                              >> /etc/kibana/kibana.yml
                systemctl enable kibana
                systemctl start kibana
        SHELL
  end

end
