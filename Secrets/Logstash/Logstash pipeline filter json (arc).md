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
        match => { "message" => "^%{NUMBER:locx}\s+%{NUMBER:locy}\s+%{WORD:vddpin}\s+%{WORD:vsspin}\s+%{WORD:vdddomain}\s+%{WORD:vssdomain}\s+%{NUMBER:effvdd}\s+%{NUMBER:maxpgwt}\s+%{NUMBER:minpgtw}\s+%{NUMBER:minpgcyc}\s+%{NUMBER:maxpgcyc}\s+%{NUMBER:numberpgarcs}\s+%{DATA:instance_name}$" }
        tag_on_failure => []
    }

    # Переименовываем и удаляем ненужные поля
    mutate {
        remove_field => ["log", "event", "@version", "host", "@timestamp", "tags", "message"]
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