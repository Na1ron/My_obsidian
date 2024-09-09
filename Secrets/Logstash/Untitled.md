match => { "message" => "^{DATA:instance_name}\s+%{NUMBER:power}\s+%{WORD:power_pin}$" }




match => { "message" => "^%{NUMBER:locx}\s+%{NUMBER:locy}\s+%{WORD:vddpin}\s+%{WORD:vsspin}\s+%{WORD:vdddomain}\s+%{WORD:vssdomain}\s+%{NUMBER:effvdd}\s+%{NUMBER:maxpgwt}\s+%{NUMBER:minpgtw}\s+%{NUMBER:minpgcyc}\s+%{NUMBER:maxpgcyc}\s+%{NUMBER:numberpgarcs}\s+%{DATA:instance_name}$" }