```java
@Slf4j
@Component
@Order(1)
public class JwtFilter implements Filter {

    @Override
    public void destroy() {
        log.info("===== filter destory =====");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)

        throws IOException, ServletException {

//        System.out.println("===== before(filter) =====");
        HttpRequestWithModifiableParameters modifyRequest = new HttpRequestWithModifiableParameters((HttpServletRequest)request);

        chain.doFilter(modifyRequest, response);

//        System.out.println("===== after(filter) =====");
    }

    @Override
    public void init(FilterConfig config) throws ServletException {
        log.info("===== init filter - This Service use Filter =====");
    }

}

## 
```java
public class HttpRequestWithModifiableParameters extends HttpServletRequestWrapper {

    private HashMap<String, Object> params;
    private byte[] bodyData;

    @SuppressWarnings("unchecked")
    public HttpRequestWithModifiableParameters(HttpServletRequest request) throws IOException {
        super(request);
        this.params = new HashMap<>(request.getParameterMap());

        // header 에 spiderman 에서 보낸 정보에서 auth 정보 취득
        Map<String, String> jwtResult;
        Map<String, Object> paramMap = new HashMap<>();
        paramMap.put("mbr_id","Y");
        paramMap.put("cust_co_cd", "Y");
        jwtResult = JwtUtil.jwtToJsonConvert(paramMap);
        String mbrId = MapUtils.getString(jwtResult,"mbr_id");
        String custCoCd = MapUtils.getString(jwtResult,"cust_co_cd");

        // 개발 편의성을 위해 임시로 헤더에서 적접 받는 로직도 남겨둔다.
        if(StringUtils.isEmpty(mbrId)){
            mbrId = request.getHeader("mbrId");
        }
        if(StringUtils.isEmpty(custCoCd)){
            custCoCd = request.getHeader("custCoCd");
        }

        // request body 를 변경 하는 부분,
        InputStream is = super.getInputStream();
        bodyData = IOUtils.toByteArray(is);
        String requestStringBody = new String(bodyData);

        if(!StringUtils.isEmpty(requestStringBody)){
            if(requestStringBody.substring(0,1).equals("{")){ // list 등의 케이스 예외처리
                JSONObject json = new JSONObject(requestStringBody);
                json.put("mbrId",mbrId);
                json.put("custCoCd",custCoCd);
                bodyData = json.toString().getBytes();
            }
        }

        setParameter("mbrId",mbrId);
        setParameter("custCoCd",custCoCd);

    }

    public String getParameter(String name) {
        String returnValue = null;
        String[] paramArray = getParameterValues(name);
        if (paramArray != null && paramArray.length > 0) {
            returnValue = paramArray[0];
        }
        return returnValue;
    }

    @SuppressWarnings("unchecked")
    public Map getParameterMap() {
        return Collections.unmodifiableMap(params);
    }

    @SuppressWarnings("unchecked")
    public Enumeration getParameterNames() {
        return Collections.enumeration(params.keySet());
    }

    public String[] getParameterValues(String name) {
        String[] result = null;
        String[] temp = (String[]) params.get(name);
        if (temp != null) {
            result = new String[temp.length];
            System.arraycopy(temp, 0, result, 0, temp.length);
        }
        return result;
    }

    public void setParameter(String name, String value) {
        String[] oneParam = { value };
        setParameter(name, oneParam);
    }

    public void setParameter(String name, String[] value) {
        params.put(name, value);
    }

    public ServletInputStream getInputStream() throws IOException {

        final ByteArrayInputStream bis = new ByteArrayInputStream(bodyData);



        return new ServletInputStreamImpl(bis);

    }



    class ServletInputStreamImpl extends ServletInputStream {

        private InputStream is;



        public ServletInputStreamImpl(InputStream bis){

            is = bis;

        }



        public int read() throws IOException {

            return is.read();

        }



        public int read(byte[] b) throws IOException {

            return is.read(b);

        }

        @Override public boolean isFinished() {
            return false;
        }

        @Override public boolean isReady() {
            return false;
        }

        @Override public void setReadListener(ReadListener readListener) {

        }
    }



}
```
