---
title: java_restApi
categories:
  - java
tags:
  - rest
  - api
date: 2020-08-11 13:41:40
thumbnail:
---

JAVA REST 송 수신 방법

## 송신

### 코드

``` bash
	// 서버요청(파람, URL 정보설정)
	public JSONObject sendSMS(NaverCloudPlatformVO paramVO) throws Exception {
		JSONObject resultJson = new JSONObject();
		JSONObject returnJson = new JSONObject();
		CommonUtil.getReturnCodeFail(resultJson);
		
		logger.debug("paramVO:"+ paramVO.toStringVo());
		
		Map<String, Object> reqMap = new HashMap<String, Object>();
		Map<String, Object> dataMap = new HashMap<String, Object>();
		
		try {
						
			dataMap.put("contentType", paramVO.getContentType());
			dataMap.put("from", NAVER_SMS_SEND_NUMBER);
			dataMap.put("content", paramVO.getMessages().get(0).get("content"));
			dataMap.put("messages", paramVO.getMessages());
			dataMap.put("reserveTime", paramVO.getReserveTime());
			dataMap.put("scheduleCode", paramVO.getScheduleCode());

			reqMap.put("dataJson", UtilJson.changeObjToJson(dataMap));
			reqMap.put("requestUrl", requestUrl);
			
			// SMS 전송
			returnJson = apiSMSPost(reqMap);
			
			// 발송성공
			if("202".equalsIgnoreCase((String) returnJson.get("naverStatCode"))) {
				CommonUtil.getReturnCodeSuc(resultJson);
			}else {
				// 발송실패
			}
			// E: DB 저장
			
			resultJson.put("datas", returnJson);
			
		}catch (Exception e) {
			e.printStackTrace();
			CommonUtil.getReturnCodeFail(resultJson, e.toString());
		}
		
		return resultJson;
	}



	// 서버통신
	public JSONObject apiSMSPost(Map<String, Object> reqMap) throws Exception {
		JSONObject resultJson = new JSONObject();
		CommonUtil.getReturnCodeFail(resultJson);
		
		//http client 생성
		RequestConfig.Builder config = RequestConfig.custom();
		config.setConnectTimeout(conTimeout * 1000).setConnectionRequestTimeout(reqTimeout * 1000).setSocketTimeout(socTimeout * 1000);
		HttpClientBuilder builder = HttpClientBuilder.create();
		builder.setDefaultRequestConfig(config.build());
		CloseableHttpClient httpClient = builder.build();
		
		try{
			
			logger.debug("[post]requestUrl:"+ (String) reqMap.get("requestUrl"));
			
			HttpPost httpClientRequest = new HttpPost(NAVER_SMS_API_URL + (String) reqMap.get("requestUrl")); //POST 메소드 URL 새성 
			
			//makeSignature 정보 설정
			String ts = String.valueOf(CommonUtil.getTimestamp());
			NaverCloudPlatformVO msVO = new NaverCloudPlatformVO();
			msVO.setMsMethod("POST");
			msVO.setMsTimestamp(ts);
			msVO.setMsUrl("/sms/v2"+ (String) reqMap.get("requestUrl"));
			
			httpClientRequest.addHeader("Content-Type", "application/json;charset=UTF-8");
			httpClientRequest.addHeader("x-ncp-apigw-timestamp", ts);
			httpClientRequest.addHeader("x-ncp-iam-access-key", NAVER_ACCESS_KEY_ID);
			httpClientRequest.addHeader("x-ncp-apigw-signature-v2", makeSignature(msVO));
			
			StringEntity inputParam = new StringEntity(reqMap.get("dataJson").toString(), "UTF-8");
			inputParam.setContentType("application/json");
			httpClientRequest.setEntity(inputParam);
			
			CloseableHttpResponse httpResponse = httpClient.execute(httpClientRequest);
			logger.debug("POST Response Status");
			logger.debug(String.valueOf(httpResponse.getStatusLine().getStatusCode()));
			String jsonStr = EntityUtils.toString(httpResponse.getEntity(), "UTF-8");
			
			if(!"202".equalsIgnoreCase(String.valueOf(httpResponse.getStatusLine().getStatusCode()))) {
				if(CommonUtil.isNotEmpty(httpResponse.getEntity())) {
					jsonStr = EntityUtils.toString(httpResponse.getEntity(), "UTF-8");
				}
			}else {
				CommonUtil.getReturnCodeSuc(resultJson);
			}
			resultJson = UtilJson.changeStringToJson(jsonStr);
			resultJson.put("naverStatCode", String.valueOf(httpResponse.getStatusLine().getStatusCode()));
			
			logger.debug("POST Request\n"+reqMap.get("dataJson"));
			logger.debug("RESULT:\n"+jsonStr);
		}catch (Exception e) {
			e.printStackTrace();
			CommonUtil.getReturnCodeFail(resultJson, e.toString());
		}finally {
			try {
				httpClient.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		
		return resultJson;
	}
```

## 수신

### 코드

``` bash
	@RequestMapping(value="/myContractList", produces="application/json;charset=utf8")
	public JSONObject myContractList(@RequestBody Map<String, Object> paramMap, HttpSession session) {
		
		logger.info("앱)나의체결현황 조회");
		
		JSONObject resultJson = new JSONObject();
		CommonUtil.getReturnCodeFail(resultJson);
		
		if(CommonUtil.isEmpty(paramMap.get("CLNT_NO"))
				|| CommonUtil.isEmpty(paramMap.get("MOBILE_TOCKEN"))
				){
			CommonUtil.getReturnCodeFailMobileToken(resultJson, MSG_TOKEN_ERROR);
			return resultJson;
		}
		
		if(CommonUtil.isEmpty(session.getAttribute("M_USER_"+ (String) paramMap.get("CLNT_NO")))) {
			CommonUtil.getReturnCodeFailSession(resultJson, MSG_SESSION_ERROR);
			return resultJson;
		}
		
		// 결과값 가공할거임(앱으로 안내려감)
		List<InvstMyPageVO> resultList = new ArrayList<InvstMyPageVO>();
		
		Map<String, Object> datasMap = new HashMap<String,Object>();
		// 실제 앱에 내려감
		List<Map<String, Object>> myContractList = new ArrayList<Map<String,Object>>();
		Map<String, Object> myContract = new HashMap<String, Object>();
		int resultCount = 0;
		
		InvstMyPageVO paramVO = new InvstMyPageVO();
		
		
		try {
			// 토큰확인
			OmpLoginVO param_token = new OmpLoginVO();
			param_token.setMOBILE_ID((String)paramMap.get("CLNT_NO"));
			param_token.setTOKEN_VALUE((String)paramMap.get("MOBILE_TOCKEN"));
			int tokenCnt = ompLoginService.selectChkMobTokenCnt(param_token);
			if(tokenCnt <= 0) {
				CommonUtil.getReturnCodeFailMobileToken(resultJson, MSG_TOKEN_ERROR);
				return resultJson;
			}
			// 토큰확인
			
			
			if(CommonUtil.isEmpty(paramMap.get("clnt_no"))) {
				CommonUtil.getReturnCodeFail(resultJson, "필수값을 확인해주세요(clnt_no");
				return resultJson;
			}
			
			// 파람설정
			paramVO.setPBLS_CLNT_NO((String) paramMap.get("clnt_no"));
			
			// 페이징 시작번호 초기값 0
			if(CommonUtil.isEmpty(paramMap.get("start"))){
				paramVO.setStart(0);
			}else {
				paramVO.setStart((int) paramMap.get("start"));
			}
			// 페이징 마지막번호 초기값 10 
			if(CommonUtil.isEmpty(paramMap.get("end"))){
				paramVO.setEnd(10);
			}else {
				paramVO.setEnd((int) paramMap.get("end"));
			}
			
			// 조회날짜 yyyymmdd
			if(CommonUtil.isNotEmpty(paramMap.get("searchRequstDeBgn")) && CommonUtil.isNotEmpty(paramMap.get("searchRequstDeEnd"))) {
				paramVO.setSearchRequstDeBgn(CommonUtil.changeDateFormat((String) paramMap.get("searchRequstDeBgn"), "yyyyMMdd", "yyyy.MM.dd"));
				paramVO.setSearchRequstDeEnd(CommonUtil.changeDateFormat((String) paramMap.get("searchRequstDeEnd"), "yyyyMMdd", "yyyy.MM.dd"));
			}
			
			paramVO.setSTK_NM((String) paramMap.get("stk_nm"));
			// 파람설정
			
			
			resultCount = invstMyPageService.selectContMngCnt(paramVO);
			resultList = invstMyPageService.selectContMngList(paramVO);
			
			if(resultCount > 0) {
				for(InvstMyPageVO result : resultList) {
					myContract = new HashMap<String, Object>();
					myContract.put("ORDER_NO", result.getORDER_NO());			// 주문번호
					myContract.put("CHANNEL_URL", result.getCHANNEL_URL());		// 방아이디
					myContract.put("POST_TIME", result.getPOST_TIME());		// 주문게시일시
					myContract.put("STK_CODE", result.getSTK_CODE());		// 종목코드
					myContract.put("STK_NM", result.getSTK_NM());		// 종목명
					myContract.put("STOCK_TP_CODE_NM", result.getSTOCK_TP_CODE_NM());		// 종목구분 
					myContract.put("STK_NM_FULL", result.getSTK_NM_FULL());		// 종목명 풀
					myContract.put("DEAL_QTY", result.getDEAL_QTY());		// 협상수량
					myContract.put("DEAL_UPRC", result.getDEAL_UPRC());		// 협상단가
					myContract.put("DEAL_TOT", result.getDEAL_UPRC());		// 협상금액
					myContract.put("DEAL_TP", result.getDEAL_TP());		// 주문구분(10:매도, 20:매수)
					myContract.put("NEGO_REQST_CLNT_NM", result.getNEGO_REQST_CLNT_NM());		// 거래상대방 이름(마스킹처리됨) 사용안함
					myContract.put("NEGO_REQST_CLNT_NICKNAME", result.getNEGO_REQST_CLNT_NICKNAME());		// 거래상대방 닉네임(이거로 표시)
					myContract.put("NEGO_SETT_STAT_CODE", result.getNEGO_SETT_STAT_CODE());		// 협상상태코드
					myContract.put("RQST_CODE", result.getRQST_CODE());		// 내주문 구문(0:내가주문게시자인 협상 1:내가 협상 요청자인 협상)
					myContract.put("STAT_FLAG", result.getSTAT_FLAG());		// 협상상태(서명대기,입금대기,취소,체결,완료)
					myContract.put("CONT_FLAG", result.getCONT_FLAG());		// 전자서명여부(Y:서명완료, N:미서명)
					myContract.put("ESCR_FLAG", CommonUtil.emptyRstr(result.getESCR_FLAG()));		// 입금상태명(매수대금입금대기, 매수대금입금실패, 매도계좌이체대기,매도계좌이체완료,매도계좌이체실패,환불처리중,환불처리완료,환불처리실패)
					myContract.put("ESCR_STATUS_FLAG", CommonUtil.emptyRstr(result.getESCR_STATUS_FLAG()));		// 입금상태(300:입금대기, 311:입금실패[테스트:999999999, 상용:BEBK20000], 320:입금완료, 330:이체지시, 331:이체결과(성공[테스트:000000000, 상용:NCOM00000], 실패[그외]),340:환불지시,34:환불완료(성공실패모두)
					myContract.put("TRAN_FLAG", CommonUtil.emptyRstr(result.getTRAN_FLAG()));		// 기업승인여부(Y:승인, N:미승인, '' 승인대상이아님)
					myContract.put("TRNS_ID", CommonUtil.emptyRstr(result.getTRNS_ID()));		// 계약서 txID
					myContract.put("CHG_DTTM", result.getCHG_DTTM());		// 협상변경일시
					myContract.put("SEC_KIND_DTL_TP_CODE", result.getSEC_KIND_DTL_TP_CODE());		// 증권종류상세구분코드 
					myContract.put("CORP_HANGL_NM", result.getCORP_HANGL_NM());		// 기업명 
					myContract.put("RMQTY", result.getRMQTY());		// 주문잔량 
					myContract.put("PBLS_CLNT_NO", result.getPBLS_CLNT_NO());		// 주문게시자고객번호 
					myContract.put("ERR_CODE", CommonUtil.emptyRstr(result.getERR_CODE()));		// 에스크로에러코드 
					myContract.put("ERR_RSN", CommonUtil.emptyRstr(result.getERR_RSN()));		// 에스크로에러사유 
					myContract.put("PUBLIC_YN", result.getPUBLIC_YN());		// 공개주문여부(Y:공개 N:비공개주문)
					myContract.put("CERTI_NUM", CommonUtil.emptyRstr(result.getCERTI_NUM()));		// 비공개주문인증번호
					myContract.put("IS_MY_ORDER", CommonUtil.emptyRstr(result.getIS_MY_ORDER()));		// 내주문여부(Y:내꺼 N:남의꺼)
					myContractList.add(myContract);
				}
			}
			
			
			datasMap.put("resultCount", resultCount);
			datasMap.put("resultList", myContractList);
			
			CommonUtil.getReturnCodeSuc(resultJson);
			
		} catch (Exception e) {
			e.printStackTrace();
			CommonUtil.getReturnCodeFail(resultJson, e.toString());
		}
		
		resultJson.put("params", paramMap);
		resultJson.put("datas", datasMap);
		
		logger.info("resultJson:" + resultJson.toString());
		return resultJson;
	}
```