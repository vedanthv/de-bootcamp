# DataExpert.io DE Bootcamp

## Troubleshooting

1. How to ```git clone...```

Check this : https://github.com/orgs/community/discussions/101536

![image](https://github.com/user-attachments/assets/bbae39ab-6997-4f41-9307-a95258288581)

2. Creating the first set of tables.

Idea is to create the table in the docker container volume so that its persistent.

Commands in the official README doesnt seem to work...

Workaround : 

**docker_compose.yml**

```
version: "3.10.6"
services:
  postgres:
    image: postgres:14
    restart: on-failure
    container_name: ${DOCKER_CONTAINER}
    env_file:
      - .env
      - example.env
    environment:
      - POSTGRES_DB=${POSTGRES_SCHEMA}
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    ports:
      - "${CONTAINER_PORT}:5432"
    volumes:
      - ./:/bootcamp/
      - ./data.dump:/docker-entrypoint-initdb.d/data.dump
      - ./scripts/init-db.sh:/docker-entrypoint-initdb.d/init-db.sh
      - postgres-data:/var/lib/postgresql/data
volumes:
  postgres-data:
```

**scrips/init-db.sh**

```
#!/bin/bash
set -e

# Restore the dump file using pg_restore
pg_restore \
    -v \
    --no-owner \
    --no-privileges \
    -U $POSTGRES_USER \
    -d $POSTGRES_DB \
    /docker-entrypoint-initdb.d/data.dump

# Check if the path is a directory using the -d flag and
#  there are SQL files in the directory using the -f command
#   (the [] brackets are used for conditional expressions)
if [ -d /docker-entrypoint-initdb.d/homework ]; then
  echo "[SUCCESS]: Located homework directory"
  # Run any additional initialization scripts
    for f in /docker-entrypoint-initdb.d/homework/*.sql; do
      if [ -f "$f" ]; then
        echo "[SUCCESS] Running SQL file: $f"
        psql -U $POSTGRES_USER -d $POSTGRES_DB -f $f
      else
        echo "[INFO] No SQL file found inside the homework directory"
      fi
    done
else
    echo "[ERROR] Directory not found: /docker-entrypoint-initdb.d/homework/"
fi
```

**Commands to Run**

```docker exec -it <container_name_or_id> bash```

```pg_restore -U $POSTGRES_USER -d $POSTGRES_DB /docker-entrypoint-initdb.d/data.dump```

The data would persist in volume even if we do ```docker compose down```

**Show the data**

![image](https://github.com/user-attachments/assets/91036353-0b0d-4b73-a9af-3fa87017d776)

