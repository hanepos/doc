# 土台となるマイクロサービスの作成
## 実施したこと概要
* [AWSで作るマイクロサービス](https://news.mynavi.jp/itsearch/series/devsoft/aws_2.html)の第3回〜第7回
## 実施したこと詳細
### フロントエンド側プロジェクト作成
Spring Securityを使用してログイン処理を行う単一プロジェクトを作成<br>
詳細は[こちら](spring-security.html)
### バックエンドサービス作成
DBに格納しているユーザ情報を取得するバックエンドサービスを作成<br>
[AWSで作るマイクロサービス](https://news.mynavi.jp/itsearch/series/devsoft/aws_2.html)の第5回
* 記事以外で実施したこと
  - Mavenマルチプロジェクト化
  - commonプロジェクト作成

* つまったところ
  - とくになし（JPAむずかしい、、）

### フロントエンド側プロジェクトをバックエンドサービスと連携
ログインの際にバックエンドサービスと連携してユーザ認証を行うように修正<br>
[AWSで作るマイクロサービス](https://news.mynavi.jp/itsearch/series/devsoft/aws_2.html)の第6,7回
* 記事以外で実施したこと
  - Spring Boot2.4.0（Spring5.3？）で実装したため、HandlerInterceptorAdaptorがdeprecatedになっていたおり、HandlerInterceptor(preHandle,postHandle,afterCompletionがdefaultメソッドとして提供)を実装するように変更

     ```Java
      public class SetMenuInterceptor implements HandlerInterceptor {
          @Override
          public void postHandle(HttpServletRequest request,
                                 HttpServletResponse response,
                                 Object handler, 
                                 ModelAndView modelAndView) throws Exception {
                // ommit;
            }
       } 
     ```

* つまったところ
  - pom.xmlに依存性（spring-web-flux）追加時に変更が反映されない  
⇒frontendがMavenプロジェクトと認識されず反映されなかったため、  
プロジェクトを右クリック→Add Framework supportしてから  
pom.xmlを右クリック→Maven→Reimportを実施
  - lombockはbooleanだとgetterがis〜になるが､Booleanの場合はget~になる（デフォルトの場合）
  - CustomUserDetailsService内でAutowiredしているOrchestrationServiceのインスタンスがnullでぬるぽ発生→CustomUserDetailsService内がBean登録されていなかった、、、
