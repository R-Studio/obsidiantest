# YAML
## Bash-only Function to parse YAML using SED / AWK
```bash
function parse_yaml {
   local prefix=$2
   local s='[[:space:]]*' w='[a-zA-Z0-9_]*' fs=$(echo @|tr @ '\034')
   sed -ne "s|^\($s\):|\1|" \
        -e "s|^\($s\)\($w\)$s:$s[\"']\(.*\)[\"']$s\$|\1$fs\2$fs\3|p" \
        -e "s|^\($s\)\($w\)$s:$s\(.*\)$s\$|\1$fs\2$fs\3|p"  $1 |
   awk -F$fs '{
      indent = length($1)/2;
      vname[indent] = $2;
      for (i in vname) {if (i > indent) {delete vname[i]}}
      if (length($3) > 0) {
         vn=""; for (i=0; i<indent; i++) {vn=(vn)(vname[i])("_")}
         printf("%s%s%s=\"%s\"\n", "'$prefix'",vn, $2, $3);
      }
   }'
}
```

**YAML test file (sample.yaml)**
```yaml
## global definitions
global:
  debug: yes
  verbose: no
  debugging:
    detailed: no
    header: "debugging started"

## output
output:
   file: "yes"
```

**Using the function**
```console
parse_yaml sample.yaml
```

**Output**
```ini
global_debug="yes"
global_verbose="no"
global_debugging_detailed="no"
global_debugging_header="debugging started"
output_file="yes"
```
*[Source](https://stackoverflow.com/questions/5014632/how-can-i-parse-a-yaml-file-from-a-linux-shell-script)*
