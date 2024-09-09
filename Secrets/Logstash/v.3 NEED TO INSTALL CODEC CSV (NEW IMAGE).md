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
    # Разбор строк CSV-файлов
    csv {
        columns => ["instance_name", "locx", "locy", "vdd_pin", "vss_pin", "vdd_domain", "vss_domain", "eff_vdd", "maxpgwt", "min_pg_tw", "min_pg_cyc", "max_pg_cyc", "numberpgarcs"]
        separator => ","
        skip_empty_columns => true
        target => "csv_data"
        add_field => { "source_file" => "%{type}" }
    }

    csv {
        columns => ["instance_name", "powerw", "powerpin"]
        separator => ","
        skip_empty_columns => true
        target => "csv_data"
        add_field => { "source_file" => "%{type}" }
    }
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
}

    # Убираем временные поля
    mutate {
        remove_field => ["csv_data", "source_file", "path", "@version", "@timestamp", "host", "message", "tags"]
    }
}

output {
    if "aggregated" in [tags] {
        file {
            path => "/usr/share/logstash/data/output/merge.csv"
            codec => line {
                format => "%{instance_name},%{field1},%{field2},%{field3},%{field4},%{field5},%{field6},%{field7},%{field8},%{field9},%{field10}" "%{instance_name}",locx", "locy", "vdd_pin", "vss_pin", "vdd_domain", "vss_domain", "eff_vdd", "maxpgwt", "min_pg_tw", "min_pg_cyc", "max_pg_cyc", "numberpgarcs", "powerw", "powerpin"
            }
        }
    }
}