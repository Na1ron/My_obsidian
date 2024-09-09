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
    # Вывод в консоль для отладки
    stdout {
        codec => rubydebug
    }

    # Запись в файл
    file {
        path => "/usr/share/logstash/data/output/arc.json"
        codec => json_lines
    }
}