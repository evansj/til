# Read Java properties files

This is a quick and dirty function for looking up values from java properties files. It correctly deals with values which contain embedded `${var}` references, by calling itself recursively. It won't deal with things like line continuations.

For example, when reading this entry:

    db.url=jdbc:mysql://${db.host}:${db.port}/${db.name}?useUnicode=yes&characterEncoding=UTF-8

it will look up the values of `db.host`, `db.port` and `db.name` from the same file before returning the value.

```bash
function property {
  local KEY=$1
  local FILE=$2
  local KV=$(cat $FILE | grep -E "^$KEY\s*=")
  local VALUE=$(sed -E "s/^$KEY\s*=\s*//" <<< "$KV")

  # Recursively call this function to replace any ${key}s from the string
  while [[ $VALUE =~ \$\{([^}]+)\} ]]; do
    local K=${BASH_REMATCH[1]}
    local V=$(property $K $FILE)
    VALUE=${VALUE//\$\{$K\}/$V}
  done

  echo $VALUE
}
```

Call this function with the properties filename and the key you are looking for, e.g.:

```bash
DB_URL=$(property db.url /path/to/database.properties)
```