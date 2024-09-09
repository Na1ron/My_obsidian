input {
  file {
    path => "/usr/share/logstash/data/input/stack1.csv"
    start_position => "beginning"
    sincedb_path => "null"
    id => "stack1"
  }

  file {
    path => "/usr/share/logstash/data/input/stack2.csv"
    start_position => "beginning"
    sincedb_path => "null"
    id => "stack2"
  }
}

filter {
  if [@metadata][id] == "stack1" {
    csv {
      columns => [ "PATH", "VALUE1", "VALUE2", "NET1", "NET2", "NET3", "NET4", "PARAM1", "PARAM2", "PARAM3", "PARAM4", "PARAM5", "STATUS" ]
      separator => ","
    }
    mutate {
      add_field => { "file_type" => "stack1" }
    }
  }

  if [@metadata][id] == "stack2" {
    csv {
      columns => [ "PATH", "PARAM6", "NET5" ]
      separator => ","
    }
    mutate {
      add_field => { "file_type" => "stack2" }
    }
  }

  # Aggregate and combine data from stack2 with stack1
  if [file_type] == "stack2" {
    aggregate {
      task_id => "%{PATH}"
      code => "
        map['PARAM6'] = event.get('PARAM6')
        map['NET5'] = event.get('NET5')
      "
      map_action => "create"
    }
    drop {}
  }

  if [file_type] == "stack1" {
    aggregate {
      task_id => "%{PATH}"
      code => "
        event.set('PARAM6', map['PARAM6'])
        event.set('NET5', map['NET5'])
      "
      map_action => "update"
      end_of_task => true
      timeout => 120
    }
  }
}

output {
  file {
    path => "/usr/share/logstash/data/output/stack3.txt"
    codec => line {
      format => "%{PATH},%{VALUE1},%{VALUE2},%{NET1},%{NET2},%{NET3},%{NET4},%{PARAM1},%{PARAM2},%{PARAM3},%{PARAM4},%{PARAM5},%{STATUS},%{PARAM6},%{NET5}"
    }
  }
  stdout { codec => rubydebug }
}
