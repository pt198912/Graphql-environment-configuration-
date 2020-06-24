# Graphql-environment-configuration-
1、在使用graphql的主module的build.gradle添加

    apply plugin: 'com.apollographql.android'（需写在apply plugin: 'com.android.application'这行以下），不能写在library形式的module    中，这样会无法自动编译.graphql文件生成相关的接口api类

2、在主module的graphql中添加

    repositories {
      jcenter()
    }

    dependencies {
      implementation("com.apollographql.apollo:apollo-runtime:x.y.z")

      // optional: if you want to use the normalized cache
      implementation("com.apollographql.apollo:apollo-normalized-cache-sqlite:x.y.z")
      // optional: for coroutines support
      implementation("com.apollographql.apollo:apollo-coroutines-support:x.y.z")
      // optional: for RxJava3 support  
      implementation("com.apollographql.apollo:apollo-rx3-support:x.y.z")

      // optional: if you just want the generated models and parser write your own HTTP code/cache code   
      implementation("com.apollographql.apollo:apollo-api:x.y.z")
    }
    
 3、在根目录的build.gradle中添加
   
    classpath 'com.apollographql.apollo:apollo-gradle-plugin:1.0.1'
    
 4、在主module的app/src/main/下创建graphql目录,在graphql目录下放入schema.json，在graphql目录下创建com.graphql.data子目录，在data子目录下   面存放.graphql接口查询文件（之所以创建com.graphql.data子目录是因为自动生成的接口查询类需要相应的包名，不然会无法引用）
    
   schema.json文件由命令行生成：
   
    1、npm install -g apollo-codegen
    2、apollo-codegen download-schema 服务器地址 --output schema.json
    如果下载schema.json时后端加入了token校验，则命令行需添加 header 参数，传token,比如
    apollo-codegen download-schema http://***/graphql/  --output schema.json  --header "Authorization":"bearer $token”）
    
   queryAllShop.graphql的语言示例如下
   
    query allShops{
      allShops{
          address imgUrl
      }
    }
    
   queryOneShop.graphql的语言示例如下
    
    query oneShop($id:Int!,$name:String!){
      oneShop(id:$id,name:$name){
          address imgUrl
      }
    }

5、graphql接口调用示例
  
     AllShopsQuery allShopsQuery = AllShopsQuery.builder()
                  .build();
     ApolloCall<AllShopsQuery.Data> allShopsQueryCall = ApolloApiManager.Companion.getInstacne().getClient()
                  .query(allShopsQuery);
     allShopsQueryCall.enqueue(new ApolloCall.Callback<AllShopsQuery.Data>() {
              @Override
              public void onResponse(@NonNull Response<AllShopsQuery.Data> response) {
                  AllShopsQuery.Data data = response.data();
                  List<AllShopsQuery.AllShop> shops=data.allShops();
                  Log.d(TAG, "apollo onResponse: "+data.allShops());
              }

              @Override
              public void onFailure(@NonNull ApolloException e) {
                  Log.d(TAG, "apollo onFailure: ");
              }
          });
          
 
 6.对于graphql中的scalar自定义数据类型的处理
 
    在app的build.gradle中添加自定义customTypeAdapter,如下代码
        apollo {
            customTypeMapping = [
                    "TimeStamp" : "java.lang.Long",
                    "Long" : "java.lang.String"
            ]
        }
