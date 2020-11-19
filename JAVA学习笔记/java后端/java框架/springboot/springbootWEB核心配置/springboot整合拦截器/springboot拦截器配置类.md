```java
public class MyInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        HttpSession session = request.getSession();
        String user = (String) session.getAttribute("user");
        if (!StringUtils.isEmpty(user)){
            return true;
        }else{
            request.setAttribute("msg","你还没有登录");
            request.getRequestDispatcher("/index.html").forward(request,response);
            return false;
        }
    }
}
```

