# django-migration-issue-sqlite-mysql

## Add `slave Mysql DB` information in `settings.py`

```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    },

    'slave': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'staging',
        'USER': 'username',
        'PASSWORD': 'xyz',
        'HOST': 'private-db.ondigitalocean.com',
        'PORT': '25060',
        'OPTIONS': {
                'init_command': "SET sql_mode='STRICT_TRANS_TABLES'",
                'charset': 'utf8mb4'
            }
    }
}
```

## Then run,

```
python manage.py migrate --database slave
```

## Possible Error,
```
django.db.utils.OperationalError: (1101, "BLOB, TEXT, GEOMETRY or JSON column 'applicants' can't have a default value")
```

## Fix of above possible error

```
Go to the migration file of the error and do monkeypatching by just adding below code snippet on the top:

from django.db.backends.mysql import schema
def skip_default(self, field):
    db_type = field.db_type(self.connection)
    return (
        db_type is not None and
        db_type.lower() in {
            'tinyblob', 'blob', 'mediumblob', 'longblob',
            'tinytext', 'text', 'mediumtext', 'longtext',
            'json',
            }
        )
schema.DatabaseSchemaEditor.skip_default=skip_default
```
##And Again Do Migration:

```
python manage.py migrate --database slave
```
