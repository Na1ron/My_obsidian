input {
    file {
        path => "/usr/share/logstash/data/output/arc.csv"
        start_position => "beginning"
        sincedb_path => "/dev/null"
        codec => csv {
            columns => ["instance_name", "locx", "locy", "vdd_pin", "vss_pin", "vdd_domain", "vss_domain", "eff_vdd", "maxpgwt", "min_pg_tw", "min_pg_cyc", "max_pg_cyc", "numberpgarcs"]
        }
        type => "file1"
    }
    file {
        path => "/usr/share/logstash/data/output/ipf.csv"
        start_position => "beginning"
        sincedb_path => "/dev/null"
        codec => csv {
            columns => ["instance_name", "powerw", "powerpin"]
        }
        type => "file2"
    }
}

filter {
    if [type] == "file1" {
        aggregate {
            task_id => "%{instance_name}"
            code => "
                map['instance_name'] = event.get('instance_name')
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
            "
            map_action => "create"
        }
    }

    if [type] == "file2" {
        aggregate {
            task_id => "%{instance_name}"
            code => "
                map['powerw'] = event.get('powerw')
                map['powerpin'] = event.get('powerpin')
                event.set('locx', map['locx'])
                event.set('locy', map['locy'])
                event.set('vdd_pin', map['vdd_pin'])
                event.set('vss_pin', map['vss_pin'])
                event.set('vdd_domain', map['vdd_domain'])
                event.set('vss_domain', map['vss_domain'])
                event.set('eff_vdd', map['eff_vdd'])
                event.set('maxpgwt', map['maxpgwt'])
                event.set('min_pg_tw', map['min_pg_tw'])
                event.set('min_pg_cyc', map['min_pg_cyc'])
                event.set('max_pg_cyc', map['max_pg_cyc'])
                event.set('numberpgarcs', map['numberpgarcs'])
                event.set('end_of_task', true)
            "
            map_action => "update"
            end_of_task => true
            push_previous_map_as_event => true
        }
    }
}

output {
    if [end_of_task] {
        file {
            path => "/usr/share/logstash/data/output/merge.csv"
            codec => line {
                format => "%{instance_name},%{locx},%{locy},%{vdd_pin},%{vss_pin},%{vdd_domain},%{vss_domain},%{eff_vdd},%{maxpgwt},%{min_pg_tw},%{min_pg_cyc},%{max_pg_cyc},%{numberpgarcs},%{powerw},%{powerpin}"
            }
        }
    }
}
