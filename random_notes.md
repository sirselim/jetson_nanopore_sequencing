# "Random" Notes

A 'scratch' file to jot things down before I decide where they are going to go.

## setting up the jetson mate

You can tweak the warning and error out levels of storage for the MinKNKOW app. This is the section from `app_conf`:

```json
        "disk_space_warnings": {
            "cereal_class_version": 0,
            "intermediate": {
                "cereal_class_version": 0,
                "minimum_space_mb": 1000,
                "warning_space_mb": 2000,
                "warning_interval_mb": 100,
                "only_enable_when_writing_reads": false
            },
            "logs": {
                "minimum_space_mb": 100,
                "warning_space_mb": 300,
                "warning_interval_mb": 1000,
                "only_enable_when_writing_reads": false
            },
            "reads": {
                "minimum_space_mb": 40000,
                "warning_space_mb": 100000,
                "warning_interval_mb": 50000,
                "only_enable_when_writing_reads": true
            },
            "queued_reads": {
                "minimum_space_mb": 5000,
                "warning_space_mb": 100000,
                "warning_interval_mb": 1000,
                "only_enable_when_writing_reads": true
            }
        },
```

I'm testing the current 'tweaks' on the NX modules in the Jetson Mate. Each microSD has ~30Gb of free space, which should be heaps for testing purposes.

```json
        "disk_space_warnings": {
            "cereal_class_version": 0,
            "intermediate": {
                "cereal_class_version": 0,
                "minimum_space_mb": 1000,
                "warning_space_mb": 2000,
                "warning_interval_mb": 100,
                "only_enable_when_writing_reads": false
            },
            "logs": {
                "minimum_space_mb": 100,
                "warning_space_mb": 300,
                "warning_interval_mb": 1000,
                "only_enable_when_writing_reads": false
            },
            "reads": {
                "minimum_space_mb": 4000,
                "warning_space_mb": 10000,
                "warning_interval_mb": 5000,
                "only_enable_when_writing_reads": true
            },
            "queued_reads": {
                "minimum_space_mb": 5000,
                "warning_space_mb": 10000,
                "warning_interval_mb": 1000,
                "only_enable_when_writing_reads": true
            }
        },
```

I also then changed the data location in `user_conf` back to `/data/` - which should mean that reads get written back to the microSD.
