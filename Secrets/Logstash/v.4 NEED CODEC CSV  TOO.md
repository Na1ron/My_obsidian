input {
    file {
        path => "/path/to/first_file.csv"
        start_position => "beginning"
        sincedb_path => "/dev/null"
        codec => csv {
            columns => ["instance_name", "locx", "locy", "vdd_pin", "vss_pin", "vdd_domain", "vss_domain", "eff_vdd", "maxpgwt", "min_pg_tw", "min_pg_cyc", "max_pg_cyc", "numberpgarcs"]
        }
        type => "file1"
    }

    file {
        path => "/path/to/second_file.csv"
        start_position => "beginning"
        sincedb_path => "/dev/null"
        codec => csv {
            columns => ["instance_name", "powerw", "powerpin"]
        }
        type => "file2"
    }
}

filter {
    # Добавляем поле для определения источника данных
    mutate {
        add_field => { "source_file" => "%{type}" }
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
        timeout => 24000
        timeout_tags => ['aggregated']
        timeout_code => "event.set('tags', 'aggregated')"
    }

    # Убираем временные поля
    mutate {
        remove_field => ["source_file", "path", "@version", "@timestamp", "host", "message", "tags"]
    }
}

output {
    if "aggregated" in [tags] {
        file {
            path => "/path/to/output/combined_output.csv"
            codec => csv {
                fields => ["instance_name", "locx", "locy", "vdd_pin", "vss_pin", "vdd_domain", "vss_domain", "eff_vdd", "maxpgwt", "min_pg_tw", "min_pg_cyc", "max_pg_cyc", "numberpgarcs", "powerw", "powerpin"]
            }
        }
    }
}
