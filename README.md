# text
```
((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest()
```


























```
@Aspect
@Order(90)
public class AccessLogAspect {

	private RequestResponseHolder requestResponseHolder = new DefaultRequestResponseHolder();
	private static final Logger LOGGER = LoggerFactory.getLogger(AccessLogAspect.class);
	private static ObjectMapper objectMapper = new ObjectMapper();

	/**
	 * 拦截所有@ResponseBody
	 * 
	 * @author Zhu
	 * @date 2015-5-18上午11:32:26
	 * @description
	 */
	@Pointcut("execution(* com.dianwoba..*.web.controller..*.*(..)) && @annotation(com.dianwoba.wireless.response.annotation.ResponseJson)")
	public void responseBodyPointCut() {
	}

	@AfterReturning(value = "responseBodyPointCut()", returning = "ret")
	public void afterReturning(Object ret) {
		HttpServletRequest request = requestResponseHolder.getRequest();
		StringBuffer param = new StringBuffer();
		Enumeration<String> e = request.getParameterNames();
		while (e.hasMoreElements()) {
			String key = e.nextElement();
			param.append(key).append("=").append(request.getParameter(key));
			if (e.hasMoreElements()) {
				param.append("&");
			}
		}
		String result;
		try {
			result = objectMapper.writeValueAsString(ret);
		} catch (JsonProcessingException e1) {
			LOGGER.error("afterReturning occors error", e1);
			result = "JsonProcessingException";
		}
		LOGGER.info("uri:{}, param:{}, result:{}", request.getRequestURI(), param.toString(), result);
	}
}
```
