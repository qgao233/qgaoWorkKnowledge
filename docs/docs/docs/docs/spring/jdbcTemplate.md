2022-5-17 17:16:41

## 1 spring的jdbcTemplate

[Spring--JdbcTemplate简介和常用API](https://blog.csdn.net/cold___play/article/details/100109445)

其中有一个方法：

```
Object queryForObject(String sql, Object[] args, int[] argTypes, RowMapper rowMapper)
```

查询给定SQL以从SQL创建预准备语句以及绑定到查询的参数列表，通过RowMapper将单个结果行映射到Java对象。

例：

**构造RowMapper**来返回一个`Admin`对象：

```
RowMapper<Admin> rowmapper = new RowMapper<>() {
        public Admin mapRow(ResultSet rs, int index) throws SQLException {
            int id=rs.getInt("id");
            String name = rs.getString("name");
            String pwd = rs.getString("password");
            return new Admin(id, name, pwd);
        }
    };

String sql = "select * from admin_info where id = ?";
JdbcTemplate jdbcTemplate = getCtx().getBean("jdbcTemplate", JdbcTemplate.class);
Admin admin = jdbcTemplate.queryForObject(sql, new Object[]{ 100 }, rowmapper);

System.out.println(admin); 
```

原理很简单，`jdbcTemplate.queryForObject`会直接回调你实现的`mapRow`方法，
