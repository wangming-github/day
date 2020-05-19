@CookieValue
```java
/**
 * @description 验证用户信息
 * @author wangming
 * @date 2020/5/9 15:14
 */
@GetMapping("verify") //直接获取cookie中的token值
public ResponseEntity<UserInfo> verifyUser(@CookieValue("LY_TOKEN") String token) {
    try {
        // 获取token信息
        UserInfo userInfo = JwtUtils.getInfoFromToken(token, prop.getPublicKey());
        // 成功后直接返回
        return ResponseEntity.ok(userInfo);
    } catch (Exception e) {
        // 抛出异常，证明token无效，直接返回401
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(null);
    }
}
```



------

Cookie类setMaxAge过期时间
setMaxAge() 设置值大于0, 将cookie存储于本地磁盘, 过期后删除
setMaxAge() 设置值小于0, cookie不会保存于本地, 浏览器会话结束后, 将会删除, 经过在Mac上的chorme测试, 需要结束进程后cookie才会从内存中删除
setMaxAge() 设置值等于0, 立即删除cookie

```java
// 登录
@PostMapping("/login")
@ResponseBody
public Result login(Model model, Boolean isKeepPass, String username,
                    String pwd, HttpSession session, HttpServletRequest request,
                    HttpServletResponse response) throws Exception {

    Result result = indexService.login(username, pwd, session, request);

    //登陆成功时 记录密码
    if (isKeepPass && result.getStatus() == 0) {
        Cookie cookie = new Cookie("edr_username", username);
        cookie.setMaxAge(30 * 24 * 60 * 60);// 30天后删除
        cookie.setPath("/");
        cookie.setHttpOnly(true);
        response.addCookie(cookie);

        cookie = new Cookie("edr_pwd", pwd);
        cookie.setMaxAge(30 * 24 * 60 * 60);// 30天
        cookie.setPath("/");
        cookie.setHttpOnly(true);
        response.addCookie(cookie);

    } else if (!isKeepPass && result.getStatus() == 0) {
        Cookie cookie = new Cookie("edr_username", null);
        cookie.setMaxAge(0);//setMaxAge() 设置值等于0, 立即删除cookie
        cookie.setPath("/");
        cookie.setHttpOnly(true);
        response.addCookie(cookie);

        cookie = new Cookie("edr_pwd", null);
        cookie.setMaxAge(0);//setMaxAge() 设置值等于0, 立即删除cookie
        cookie.setPath("/");
        cookie.setHttpOnly(true);
        response.addCookie(cookie);
    }
    return result;
}
```



session.invalidate()是将session设置为失效，一般在退出时使用，但要注意的是：session失效的同时 浏览器会立即创建一个新的session的，你第一个session已经失效了 所以调用它的getAttribute方法时候一定会抛出NullPointerException的

```java
// 登出
@RequestMapping("/logout")
public String logout(HttpSession session, HttpServletResponse response) throws Exception {
    session.invalidate();
    response.sendRedirect("/loginPage");
    return null;
}
```