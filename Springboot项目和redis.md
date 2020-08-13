# Springboot项目和redis

## 8.13学习

- ### redis在项目查询中的使用语法

  ps： 主要用于查询数据的缓存，在项目的service层使用

  ```java
  @Service("poetService")
  public class PoetServiceImpl implements PoetService {
  
      public static final String CACHE_KEY_Poet = "poet:";
      public static final String CACHE_KEY_Author = "author:";
      
      
      public Map<String,Object> getAuthor(Integer id){
          Map<String,Object> map = new HashMap<>();
          ValueOperations<String,Authors> operations = redisTemplate.opsForValue();
          //缓存key
          String key = CACHE_KEY_Author+id;
          //先去redis查,如果查到直接返回,如果没有的话直接去数据库查
          Authors authors = operations.get(key);
  
          //redis中无数据,去数据库捞数据
          if(authors == null) {
              authors = authorsMapper.selectByPrimaryKey(id);
              if (authors != null) {
                  //查到之后存入redis以方便下次查找
                  operations.set(key,authors);
                  map.put("msg", "ok");
                  map.put("author", authors);
                  return map;
              }
          }
          map.put("msg", "ok");
          map.put("author", authors);
          return map;
      }
  
  }
  
  
  ```

  