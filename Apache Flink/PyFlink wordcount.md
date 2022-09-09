
```python
from pyflink.table import DataTypes, TableEnvironment, EnvironmentSettings
from pyflink.table.descriptors import Schema, OldCsv, FileSystem
from pyflink.table.expressions import lit

settings = EnvironmentSettings.new_instance().in_batch_mode().use_blink_planner().build()
t_env = TableEnvironment.create(settings)

# write all the data to one file
t_env.get_config().get_configuration().set_string("parallelism.default", "1")
t_env.connect(FileSystem().path('/tmp/input')) \
    .with_format(OldCsv().field('word', DataTypes.STRING())) \
    .with_schema(Schema().field('word', DataTypes.STRING())) \
    .create_temporary_table('mySource')

t_env.connect(FileSystem().path('/tmp/output')) \
    .with_format(OldCsv().field_delimiter('\t') \
    .field('word', DataTypes.STRING()) \
    .field('count', DataTypes.BIGINT())) \
    .with_schema(Schema() \
    .field('word', DataTypes.STRING()) \
    .field('count', DataTypes.BIGINT())) \
    .create_temporary_table('mySink')

tab = t_env.from_path('mySource')
tab.group_by(tab.word).select(tab.word, lit(1).count).execute_insert('mySink').wait()
```