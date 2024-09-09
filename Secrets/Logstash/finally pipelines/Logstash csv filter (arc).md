input {
    file {
        path => "/usr/share/logstash/data/input/newfilearc.txt"
        start_position => "beginning"
        sincedb_path => "/dev/null"
        codec => plain {
            charset => "UTF-8"
        }
    }
}

filter {
    # Исключаем строки, начинающиеся с # или *
    if [message] =~ /^\s*([#\*])/ {
        drop { }
    }

    # Пытаемся распарсить строку с учетом формата
    grok {
        match => { "message" => "^%{NUMBER:locx}\s+%{NUMBER:locy}\s+%{WORD:vdd_pin}\s+%{WORD:vss_pin}\s+%{WORD:vdd_domain}\s+%{WORD:vss_domain}\s+%{NUMBER:eff_vdd}\s+%{NUMBER:maxpgwt}\s+%{NUMBER:min_pg_tw}\s+%{NUMBER:min_pg_cyc}\s+%{NUMBER:max_pg_cyc}\s+%{NUMBER:numberpgarcs}\s+%{DATA:instance_name}$" }
        tag_on_failure => []
    }

    # Переименовываем и удаляем ненужные поля
    mutate {
        remove_field => ["log", "event", "@version", "host", "@timestamp", "tags", "message"]
    }
}

output {
    # Запись в CSV файл 13 полей вместе с inst_name
    file {
        path => "/usr/share/logstash/data/output/arc.csv"
        codec => line {
            format => "%{instance_name},%{locx},%{locy},%{vdd_pin},%{vss_pin},%{vdd_domain},%{vss_domain},%{eff_vdd},%{maxpgwt},%{min_pg_tw},%{min_pg_cyc},%{max_pg_cyc},%{numberpgarcs}"
        }
    }
}