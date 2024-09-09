1. scp user@vm_ip:/path/to/archive1.tar.gz /local/path/ - cкачать что-либо с какой-то тачки с помощьб scp

2. scp /path/to/local/file user@remote_host_ip:/path/to/remote/directory - скачать что-либо со своей тачки на виртуалку
   
   
launcher@172.27.10.100

scp gitlab-jenkins@jenkins-agent-dbg:/fs/PROJECTS/ELCT/workareas/m.bykov/cdns_flow_ELCT_soc_top/workarea/signoff_pi_flat_nov28_Release/reports/dynamic_rail/elct_soc_top.dvd.arc /home/launcher/logstash_test/input

scp /fs/PROJECTS/ELCT/workareas/m.bykov/cdns_flow_ELCT_soc_top/workarea/signoff_pi_flat_nov28_Release/reports/dynamic_rail/elct_soc_top.dvd.arc launcher@docker-02:/home/launcher/logstash_test/input
