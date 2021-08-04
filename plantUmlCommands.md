# PLANTUML

er.puml
```
@startuml erdiagram
entity "humiditys" <<T,FFAA00)>> {
    + hid [PK]
    ==
    degree
    time
    created_at
    created_by
    updated_at
    updated_by
}

entity "temperatures" <<T,FFAA00)>> {
    + tid [PK]
    ==
    degree
    time
    created_at
    created_by
    updated_at
    updated_by
}

entity "illuminances" <<T,FFAA00)>> {
    + iid [PK]
    ==
    degree
    time
    created_at
    created_by
    updated_at
    updated_by
}
@endumld
```

