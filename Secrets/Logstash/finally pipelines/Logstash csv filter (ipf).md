input {
    file {
        path => "/usr/share/logstash/data/input/newfileipf.txt"
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
        match => { "message" => "^%{DATA:instance_name}\s+%{DATA:power}\s+%{WORD:power_pin}$" }
        tag_on_failure => []
    }

    # Переименовываем и удаляем ненужные поля
    mutate {
        remove_field => ["log", "event", "@version", "host", "@timestamp", "tags", "message"]
        rename => ["power", "powerw"]
        rename => ["power_pin", "powerpin"]
    }
}

output {
    # Запись в CSV файл 3 поля вместе с inst name
    file {
        path => "/usr/share/logstash/data/output/arc.csv"
        codec => line {
            format => "%{instance_name},%{powerw},%{powerpin}"
        }
    }
}