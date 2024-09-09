input {
    file {
        path => "/usr/share/logstash/data/input/arc.csv"
        start_position => "beginning"
        sincedb_path => "/dev/null"
        codec => plain {
            charset => "UTF-8"
        }
        type => "file1"
    }

    file {
        path => "/usr/share/logstash/data/input/ipf.csv"
        start_position => "beginning"
        sincedb_path => "/dev/null"
        codec => plain {
            charset => "UTF-8"
        }
        type => "file2"
    }
}

filter {
    if [type] == "file1" {
        # Разбор строки из первого файла
        grok {
            match => { "message" => "%{DATA:instance_name},%{NUMBER:locx},%{NUMBER:locy},%{WORD:vdd_pin},%{WORD:vss_pin},%{WORD:vdd_domain},%{WORD:vss_domain},%{NUMBER:eff_vdd},%{NUMBER:maxpgwt},%{NUMBER:min_pg_tw},%{NUMBER:min_pg_cyc},%{NUMBER:max_pg_cyc},%{NUMBER:numberpgarcs}" }
        }
        mutate {
            add_field => { "source_file" => "file1" }
        }
    }

    if [type] == "file2" {
        # Разбор строки из второго файла
        grok {
            match => { "message" => "%{DATA:instance_name},%{DATA:powerw},%{WORD:powerpin}" }
        }
        mutate {
            add_field => { "source_file" => "file2" }
        }
    }

    # Аггрегация данных на основе instance_name
    aggregate {
        task_id => "%{instance_name}"
        code => "
            map['instance_name'] = event.get('instance_name')
            if event.get('source_file') == 'file1'
                map['locx'] = event.get('locx')
                map['locy'] = event.get('locy')
                map['vdd_pin'] = event.get('vdd_pin')
                map['vss_pin'] = event.get('vss_pin')
                map['vdd_domain'] = event.get('vdd_domain')
                map['vss_domain'] = event.get('vss_domain')
                map['eff_vdd'] = event.get('eff_vdd')
                map['maxpgwt'] = event.get('maxpgwt')
                map['min_pg_tw'] = event.get('min_pg_tw')
                map['min_pg_cyc'] = event.get('min_pg_cyc')
                map['max_pg_cyc'] = event.get('max_pg_cyc')
                map['numberpgarcs'] = event.get('numberpgarcs')
            else
                map['powerw'] = event.get('powerw')
                map['powerpin'] = event.get('powerpin')
            end
            event.cancel()
        "
        push_map_as_event_on_timeout => true
        timeout => 60
        timeout_tags => ['aggregated']
        timeout_code => "event.set('tags', 'aggregated')"
    }

    # Убираем временные поля
    mutate {
        remove_field => ["source_file", "path", "@version", "@timestamp", "host", "message", "tags"]
    }
}

output {
    # Отладочный вывод в консоль
    stdout {
        codec => rubydebug
    }

    if "aggregated" in [tags] {
        file {
            path => "/usr/share/logstash/data/output/merge.csv"
            codec => line {
                format => "%{instance_name},%{locx},%{locy},%{vdd_pin},%{vss_pin},%{vdd_domain},%{vss_domain},%{eff_vdd},%{maxpgwt},%{min_pg_tw},%{min_pg_cyc},%{max_pg_cyc},%{numberpgarcs},%{powerw},%{powerpin}"
            }
        }
    }
}

