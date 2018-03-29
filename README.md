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
	 * @date 2015-5-18上午11:32:26
	 * @description
	 */
	@Pointcut("execution(* com.xxx..*.web.controller..*.*(..)) && @annotation(com.dianwoba.wireless.response.annotation.ResponseJson)")
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
















```
public class CityCacheService {
	private Logger logger = LoggerFactory.getLogger(CityCacheService.class);

	@Resource
	private ConfigurableEnvironment env;
	
	@Resource
	private CitysInitalizing citysInitalizing;
	private static Map<Integer, City> cache ;
	private static final String prefix = "city-";
	private static final int prefixLength = prefix.length();

	public City getCityFromCache(Integer cityId) {
		return cache.get(cityId);
	}
	
	public void reload() {
		citysInitalizing.load();
	}

	@Service
	class CitysInitalizing implements InitializingBean {

		@Override
		public void afterPropertiesSet() throws Exception {
			load();
		}

		private void load() {
			Map<Integer, City> cacheTemp = new HashMap<>();
			PropertySource<?> propertySource = env.getPropertySources().get("application");
			if (propertySource instanceof PropertiesPropertySource) {
				PropertiesPropertySource propertiesPropertySource = (PropertiesPropertySource) propertySource;
				Map<String, Object> source = propertiesPropertySource.getSource();
				for (String key : source.keySet()) {
					if (null == key) {
						continue;
					}
					if (key.startsWith(prefix)) {
						String cityInfo = (String) source.get(key);
						if (StringUtils.isNotBlank(cityInfo)) {
							String[] split = cityInfo.split(",");
							if (null != split && split.length > 1) {
								cacheTemp.put(Integer.valueOf(key.substring(prefixLength )),
										new City(split[0], split[1]));
							}
						}
					}
				}
				cache = cacheTemp;
			}
			logger.info("citys cache:{}", cache);
		}
	}
}
```
