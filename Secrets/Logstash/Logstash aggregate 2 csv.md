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
        columns => ["instance_name", "field1", "field2", "field3", "field4", "field5"]
        separator => ","
        skip_empty_columns => true
        target => "csv_data"
        add_field => { "source_file" => "%{type}" }
    }

    csv {
        columns => ["instance_name", "field6", "field7", "field8", "field9", "field10"]
        separator => ","
        skip_empty_columns => true
        target => "csv_data"
        add_field => { "source_file" => "%{type}" }
    }

    # Объединение данных по полю instance_name
    aggregate {
        task_id => "%{instance_name}"
        code => "
            map['instance_name'] = event.get('instance_name')
            if event.get('source_file') == 'file1'
                map['field1'] = event.get('[csv_data][field1]')
                map['field2'] = event.get('[csv_data][field2]')
                map['field3'] = event.get('[csv_data][field3]')
                map['field4'] = event.get('[csv_data][field4]')
                map['field5'] = event.get('[csv_data][field5]')
            else
                map['field6'] = event.get('[csv_data][field6]')
                map['field7'] = event.get('[csv_data][field7]')
                map['field8'] = event.get('[csv_data][field8]')
                map['field9'] = event.get('[csv_data][field9]')
                map['field10'] = event.get('[csv_data][field10]')
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
        remove_field => ["csv_data", "source_file", "path", "@version", "@timestamp", "host", "message", "tags"]
    }
}

output {
    if "aggregated" in [tags] {
        file {
            path => "/usr/share/logstash/data/output/merge.csv"
            codec => line {
                format => "%{instance_name},%{field1},%{field2},%{field3},%{field4},%{field5},%{field6},%{field7},%{field8},%{field9},%{field10}"
            }
        }
    }
}
