


```
package dyna.plmetc.spring.rest.report;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.HashMap;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;

import javax.servlet.http.HttpServletResponse;

import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Workbook;
import org.junit.experimental.categories.Category;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import com.fasterxml.jackson.annotation.JsonProperty.Access;

import dyna.framework.SvServer;
import dyna.framework.service.DOS;
import dyna.framework.service.ServiceID;
import dyna.framework.service.dos.DOSChangeable;
import dyna.plmetc.bom.DashboardCommonUtil;
import dyna.plmetc.bom.PartCommonUtil;
import dyna.plmetc.bom.PartInfoDTO;
import dyna.plmetc.bom.VaultCommonUtil;
import dyna.plmetc.bom.elvInfoDTO;
import dyna.plmetc.ebom.model.EBomPart;
import dyna.plmetc.spring.rest.common.BaseRestController;
import dyna.plmetc.spring.rest.common.auth.Auth;
import dyna.plmetc.spring.rest.common.auth.AuthType;
import dyna.plmetc.spring.service.pid.CostPidService;
import dyna.plmetc.spring.service.pid.PidConsts;
import dyna.plmetc.spring.service.pid.PidDesignService;
import dyna.plmetc.spring.service.pid.PidService;
import dyna.plmetc.spring.service.report.VaultReportService;
import dyna.plmetc.subae.model.SubaeManagerPick;
import dyna.plmetc.util.DBconnectionInfo;
import dyna.plmetc.util.StringUtil;
import dyna.plmetc.variant.Variant;
import dyna.plmetc.variant.VariantMap;

@Controller
@RequestMapping("/vault")
public class VaultReportController extends BaseRestController {

	private static final Logger logger = LoggerFactory.getLogger(VaultReportController.class);

	@Autowired
	VaultReportService vaultReportService;
	

	/**
	 * @category 영업사양 정보 추출 EXCEL
	 * @param model
	 * @param elvInfo
	 * @param version
	 * @return
	 * @throws Exception
	 */
	@RequestMapping(value = "/salesData", method = RequestMethod.GET)
	//public String exportSalesDataExcel(Model model, @RequestParam("elvInfo") String elvInfo, @RequestParam("version") String version) throws Exception {
	public String exportSalesDataExcel(Model model, @RequestParam("elvInfo") String elvInfo) throws Exception {
		//http://localhost/plmetc/vault/salesData?elvInfo=N22593L09&version=wip
		//https://plmpro.hdel.co.kr/plmetc/vault/salesData?elvInfo=TEST-559525
		//http://localhost/plmetc/vault/salesData?elvInfo=TEST-559525
		//http://10.105.1.45:8081/plmetc/vault/salesData?elvInfo=TEST-559525
		//TEST-559525
		logger.info("VaultReportController.exportBomDataExcel = {}", elvInfo);

		HashMap<String, String> elvInfoMap = PartCommonUtil.getLatestElvInfoOid(elvInfo);
		
		String latestOid 	 = "";
		String latestVersion = "";
		
		if(elvInfoMap != null) {
			latestOid 	 = elvInfoMap.get("OUID");
			latestVersion = elvInfoMap.get("VERSION");
		}
		
		Workbook workbook = vaultReportService.exportExcel_salesData(elvInfo, latestOid, latestVersion);

		model.addAttribute("workbook", workbook);
		//model.addAttribute("workbookName", elvInfo+ "_" + latestVersion + "_" + System.currentTimeMillis());
		model.addAttribute("workbookName", elvInfo+ "_" + latestVersion);

		return "excelDownloadView";
	}
	
	/**
	 * @category BOM 1레벨 추출
	 * @param model
	 * @param product
	 * @return
	 * @throws Exception
	 */
	@RequestMapping(value = "/bomData", method = RequestMethod.GET)
	public String exportBomDataExcel(Model model, @RequestParam("product") String product) throws Exception {
		logger.info("VaultReportController.exportBomDataExcel = {}", product);
		
		String version = "";
		//https://plmpro.hdel.co.kr/plmetc/vault/bomData?product=206397L06
		//http://localhost/plmetc/vault/bomData?product=206397L06
		
		//	http://plmtest.hdel.co.kr:8081/plmetc/vault/salesData?elvInfo=[호기번호]&version=[버전]
		//QA: http://10.105.1.45:8081/plmetc/vault/salesData?elvInfo=N22593L09&version=wip

		String latestOid	=	PartCommonUtil.getLatestProductOid(product);
		DOSChangeable productInfo = PartCommonUtil.getProductInfo(latestOid);
		
		version = (String) productInfo.get("vf$version");
		
		Workbook workbook = vaultReportService.exportExcel_bomData(product, latestOid, version, productInfo);

		model.addAttribute("workbook", workbook);
		//model.addAttribute("workbookName", product+ "_" + version + "_" + System.currentTimeMillis());
		model.addAttribute("workbookName", product+ "_" + version);

		return "excelDownloadView";
	}
	
	
	@RequestMapping(value = "/findFloorPart", method = RequestMethod.GET)
	@ResponseBody
	public HashMap<String, String> findFloorPart(Model model, @RequestParam("product") String product) throws Exception {
		logger.info("VaultReportController.findFloorPart = {}", product);
		
		String version = "";
		//https://plmpro.hdel.co.kr/plmetc/vault/bomData?product=206397L06
		//http://localhost/plmetc/vault/bomData?product=206397L06
		
		//	http://plmtest.hdel.co.kr:8081/plmetc/vault/salesData?elvInfo=[호기번호]&version=[버전]
		//QA: http://10.105.1.45:8081/plmetc/vault/salesData?elvInfo=N22593L09&version=wip

		/*
		 * String latestOid = PartCommonUtil.getLatestProductOid(product); DOSChangeable
		 * productInfo = PartCommonUtil.getProductInfo(latestOid);
		 * 
		 * version = (String) productInfo.get("vf$version");
		 * 
		 * Workbook workbook = vaultReportService.exportExcel_bomData(product,
		 * latestOid, version, productInfo);
		 * 
		 * model.addAttribute("workbook", workbook); model.addAttribute("workbookName",
		 * product+ "_" + version);
		 */
		
		HashMap<String, String> oMap = new HashMap<String, String>();
		oMap.put("NAME", "KIM");
		oMap.put("AGE", "30");
		oMap.put("COM", "HDEL");
		

		return oMap;
	}
	
	@RequestMapping(value = "/testConnect", method = RequestMethod.GET)
	@ResponseBody
	public HashMap<String, String> testConnect(Model model, String product) throws Exception {
		logger.info("VaultReportController.testConnect = {}", product);
		
		String version = "";
		//http://localhost/plmetc/vault/testConnect
		
		HashMap<String, String> oMap = new HashMap<String, String>();
		oMap.put("NAME", "KIM");
		oMap.put("AGE", "30");
		oMap.put("COM", "HDEL");
		

		return oMap;
	}
	
	
	/**
	 * @category 영업사양 정보 추출
	 * @param HogiNumber
	 * @return
	 * @throws Exception
	 */
	@RequestMapping(value = "/getSalesData", method = RequestMethod.POST)
	@ResponseBody
	public ArrayList<elvInfoDTO> getSalesData(@RequestParam String HogiNumber) throws Exception {
		
		//http://localhost/plmetc/vault/getSalesData
		//http://localhost/plmetc/vault/getSalesData?HogiNumber=N22481L03
		//plmpro.hdel.co.kr/jsp/help/salesInfoFromProductViewJson.jsp?productNumber=N22481L03&searchFlag=VERSION&=
		logger.info("VaultReportController.getSalesData = {}", HogiNumber);
		//HashMap<String, String> result = new HashMap<String, String>();
		
		ArrayList<elvInfoDTO> resultList = new ArrayList<elvInfoDTO>();
		
		//1. 호기의 최신 객체
		HashMap<String, String> hogiInfo = PartCommonUtil.getLatestElvInfoOid(HogiNumber);
		
		System.out.println("hogiInfo = " + hogiInfo);
		
		//2. 영업사양 추출
		String elvOuid = hogiInfo.get("OUID");
		
		if(elvOuid != null || !"".equalsIgnoreCase(elvOuid)) {
			resultList = PartCommonUtil.getSalesData(elvOuid);
		}

		return resultList;
	}
	
	
	/** 
	 * @category SAP JQPR 조회 
	 * @return
	 * @throws Exception
	 */
	@RequestMapping(value = "/getJQPRfromSAP", method = RequestMethod.GET)
	@ResponseBody
	public ArrayList<HashMap<String, String>> getJQPRfromSAP(String flag) throws Exception {
		
		//http://localhost/plmetc/vault/getSalesData
		//http://localhost/plmetc/vault/getSalesData?HogiNumber=N22481L03
		//plmpro.hdel.co.kr/jsp/help/salesInfoFromProductViewJson.jsp?productNumber=N22481L03&searchFlag=VERSION&=
		logger.info("VaultReportController.getJQPRfromSAP = {}");
		
		ArrayList<HashMap<String, String>> resultList = new ArrayList<HashMap<String, String>>();
		
		resultList = PartCommonUtil.getJQPRfromSAP(flag);
		
		return resultList;
	}
	
	
	/**
	 * @category JQPR 집계
	 * @return
	 * @throws Exception
	 */
	@RequestMapping(value = "/getJQPRfromSAPCount", method = RequestMethod.GET)
	@ResponseBody
	public HashMap<String, String> getJQPRfromSAPCount() throws Exception {
		
		//http://localhost/plmetc/vault/getSalesData
		//http://localhost/plmetc/vault/getSalesData?HogiNumber=N22481L03
		//plmpro.hdel.co.kr/jsp/help/salesInfoFromProductViewJson.jsp?productNumber=N22481L03&searchFlag=VERSION&=
		logger.info("VaultReportController.getJQPRfromSAPCount = {}");
		
		HashMap<String, String> resultList = new HashMap<String, String>();
		resultList = PartCommonUtil.getJQPRfromSAPCount();
		
		return resultList;
	}
	
	
	/**
	 * @category 출하예정일 (SAP : ZPPT027)
	 * @param startDate
	 * @return
	 * @throws Exception
	 */
	/*
	@RequestMapping(value = "/getZPPT027fromSAP", method = RequestMethod.POST)
	@ResponseBody
	public ArrayList<HashMap<String, String>> getZPPT027fromSAP(@RequestParam("startDate") String startDate) throws Exception {
		//http://localhost/plmetc/vault/getSalesData
		//http://10.225.4.20/plmetc/vault/getSalesData?HogiNumber=N22481L03
		//plmpro.hdel.co.kr/jsp/help/salesInfoFromProductViewJson.jsp?productNumber=N22481L03&searchFlag=VERSION&=
		logger.info("VaultReportController.getZPPT027fromSAP = {}");
		
		ArrayList<HashMap<String, String>> resultList = new ArrayList<HashMap<String, String>>();
		
		resultList = PartCommonUtil.getZPPT027fromSAP(startDate);
		
		return resultList;
	}
	*/
	
	/**
	 * @category 중국PLM에서 파트 정보 조회
	 * @param partNo
	 * @return
	 * @throws Exception
	 */
	@RequestMapping(value = "/getPartInfoWithCN", method = RequestMethod.POST)
	@ResponseBody
	public HashMap<String, String> getPartInfoWithCN(@RequestParam("partNo") String partNo) throws Exception {
		//http://localhost/plmetc/vault/getSalesData
		//http://localhost/plmetc/vault/getSalesData?HogiNumber=N22481L03
		//plmpro.hdel.co.kr/jsp/help/salesInfoFromProductViewJson.jsp?productNumber=N22481L03&searchFlag=VERSION&=
		logger.info("VaultReportController.getPartInfoWithCN = {}");
		
		System.out.println("partNo = " + partNo);
		
		HashMap<String, String> oMap = new HashMap<String, String>();
		if(partNo != null && !"".equals(partNo)) {
			oMap = PartCommonUtil.getPartInfoWithCN(partNo);			
		}
		
		return oMap;
	}
	
	
	@RequestMapping(value = "/getPartInfoWithKO", method = RequestMethod.POST)
	@ResponseBody
	public HashMap<String, String> getPartInfoWithKO(@RequestParam("partNo") String partNo) throws Exception {
		//http://localhost/plmetc/vault/getSalesData
		//http://localhost/plmetc/vault/getSalesData?HogiNumber=N22481L03
		//plmpro.hdel.co.kr/jsp/help/salesInfoFromProductViewJson.jsp?productNumber=N22481L03&searchFlag=VERSION&=
		logger.info("VaultReportController.getPartInfoWithKO = {}");
		
		System.out.println("partNo = " + partNo);
		
		HashMap<String, String> oMap = new HashMap<String, String>();
		
		if(partNo != null && !"".equals(partNo)) {
			
			String[] list = partNo.split(",");
			
			System.out.println("list = " + list.length);
			System.out.println("list = " + Arrays.toString(list));

			for(int i=0; i < list.length; i++) {
				String pNo = list[i];
				oMap = PartCommonUtil.findOneFromPartNo(pNo.trim());
			}
		}
		
		return oMap;
	}
	
	
	
	
	@RequestMapping(value = "/getPartALLWithKO", method = RequestMethod.POST)
	@ResponseBody
	public ArrayList<HashMap<String, String>> getPartALLWithKO(@RequestParam("partNo") String partNo) throws Exception {
		//http://localhost/plmetc/vault/getSalesData
		//http://localhost/plmetc/vault/getSalesData?HogiNumber=N22481L03
		//plmpro.hdel.co.kr/jsp/help/salesInfoFromProductViewJson.jsp?productNumber=N22481L03&searchFlag=VERSION&=
		logger.info("VaultReportController.getPartALLWithKO = {}");
		
		System.out.println("partNo = " + partNo);
		ArrayList<HashMap<String, String>> result = new ArrayList<HashMap<String,String>>();
		
		
		if(partNo != null && !"".equals(partNo)) {
			
			String[] list = partNo.split(",");
			
			System.out.println("list = " + list.length);
			System.out.println("list = " + Arrays.toString(list));

			for(int i=0; i < list.length; i++) {
				String pNo = list[i];
				
				HashMap<String, String> oMap = PartCommonUtil.findOneFromPartNo(pNo.trim());
				result.add(oMap);
			}
		}
		
		return result;
	}
	
	
	
	//품목별 공정진행 현황 : SAP (ZPPR030)
	@RequestMapping(value = "/searchProductIngStatus", method = RequestMethod.POST)
	@ResponseBody
	public ArrayList<HashMap<String, String>> searchProductIngStatus(String hogiNumber) throws Exception {
		
		//http://localhost/plmetc/vault/searchProductIngStatus
		//http://localhost/plmetc/vault/searchProductIngStatus?HogiNumber=N22481L03
		//plmpro.hdel.co.kr/plmetc/vault/searchProductIngStatus?HogiNumber=N22481L03
		//logger.info("VaultReportController.searchProductIngStatus = {}");
		
		ArrayList<HashMap<String, String>> resultList = new ArrayList<HashMap<String, String>>();
		
		if(hogiNumber != null && !"".equals(hogiNumber)) {
			resultList = PartCommonUtil.searchProductIngStatus(hogiNumber);
		}
		
		
		return resultList;
	}
	
	
	/**
	 * @category 호기번호로 해당 로직에디터 행 출하예정일 정보 수정
	 * @param hogiNumber
	 * @return
	 * @throws Exception
	 */
	/*
	@RequestMapping(value = "/processShipData", method = RequestMethod.GET)
	@ResponseBody
	public HashMap<String, String> processShipData(@RequestParam("hogiNumber") String hogiNumber) throws Exception {
		
		//http://localhost/plmetc/vault/getZPPT027fromHogi
		//http://localhost/plmetc/vault/processShipData?HogiNumber=N22481L03
		//plmpro.hdel.co.kr/plmetc/vault/processShipData.jsp?productNumber=N22481L03&searchFlag=VERSION&=
		logger.info("VaultReportController.processShipData = {}");

		//ArrayList<HashMap<String, String>> resultList = new ArrayList<HashMap<String, String>>();
		HashMap<String, String> result = new HashMap<String, String>();
		
		if(hogiNumber != null) {
			hogiNumber = hogiNumber.trim();
		}
		
		result = PartCommonUtil.processShipData(hogiNumber);
		
		
		return result;
	}
	*/
	
	
	/**
	 * @category 호기번호로 출하예정일 추출
	 * @param hogiNumber
	 * @return
	 * @throws Exception
	 */
	@RequestMapping(value = "/getExportDate", method = RequestMethod.POST)
	@ResponseBody
	public ArrayList<HashMap<String, String>> getExportDate(@RequestParam("hogiNumber") String hogiNumber) throws Exception {
		
		//http://localhost/plmetc/vault/getExportDate
		//http://localhost/plmetc/vault/getExportDate?hogiNumber=N22481L03
		//logger.info("VaultReportController.getExportDate = {}");

		ArrayList<HashMap<String, String>> resultList = new ArrayList<HashMap<String, String>>();
		
		if(hogiNumber != null && !"".equals(hogiNumber)) {
			resultList = PartCommonUtil.getExportDate(hogiNumber);
		}
		//result = PartCommonUtil.getZPPT027fromHogi(hogiNumber);
		return resultList;
	}
	
	
	//제품1레벨 추출
	//@Auth(authType = AuthType.NONE)
	//@Auth(authType = AuthType.ADMIN)
	@RequestMapping(value = "/getProductOneList", method = RequestMethod.GET)
	@ResponseBody
	public List<HashMap<String, String>> getProductOneList(HttpServletResponse response, @RequestParam("hogiNumber") String hogiNumber) throws Exception {
		
		//@RequestParam("authToken") String authToken
		//http://localhost/plmetc/vault/getExportDate
		//http://localhost/plmetc/vault/getProductOneList?hogiNumber=N22481L03
		//https://plmpro.hdel.co.kr/vault/getProductOneList?hogiNumber=N22481L03
		
		List<HashMap<String, String>> resultList = new ArrayList<HashMap<String, String>>();
		//List<EBomPart> resultList = new ArrayList<EBomPart>();
		
		if(hogiNumber != null && !"".equals(hogiNumber)) {
			resultList = PartCommonUtil.findDown1LevelPartV2(hogiNumber);
		}
		//result = PartCommonUtil.getZPPT027fromHogi(hogiNumber);
		
		response.setHeader("Access-Control-Allow-Origin", "*");
		response.setHeader("Access-Control-Allow-Methods", "GET,POST,DELETE,PUT,OPTIONS");
		//response.setHeader("Access-Control-Allow-Headers", "*");
		//response.setHeader("Access-Control-Allow-Credentials", "true");
		//response.setHeader("Access-Control-Max-Age", "3600");
		return resultList;
	}
	
	
	
	//부품공용화 Dashboard
	//@Auth(authType = AuthType.NONE)
	@RequestMapping(value = "/getPublicationOfPart", method = RequestMethod.GET)
	@ResponseBody
	public HashMap<String, ArrayList<HashMap<String, String>>> getPublicationOfPart(HttpServletResponse response, @RequestParam("partNo") String partNo
			, @RequestParam("specList") String specList, @RequestParam("start_date_day") String start_date_day, @RequestParam("end_date_day") String end_date_day) throws Exception {
		
		//http://localhost/plmetc/vault/getPublicationOfPart
		//http://localhost/plmetc/vault/getPublicationOfPart?hogiNumber=N22481L03
		//logger.info("VaultReportController.getExportDate = {}");
		
		HashMap<String, ArrayList<HashMap<String, String>>> resultList = new HashMap<String, ArrayList<HashMap<String, String>>>();
		if(partNo != null && !"".equals(partNo)) {
			resultList = PartCommonUtil.getPublicationOfPart(partNo, specList, start_date_day, end_date_day);
		}
		response.setHeader("Access-Control-Allow-Origin", "*");
		response.setHeader("Access-Control-Allow-Methods", "GET,POST,DELETE,PUT,OPTIONS");
		return resultList;
	}
	
	@RequestMapping(value = "/getPublicationOfPartData", method = RequestMethod.GET)
	@ResponseBody
	public ArrayList<HashMap<String, String>> getPublicationOfPartData(HttpServletResponse response, @RequestParam("partNo") String partNo, @RequestParam("specList") String specList, 
			@RequestParam("start_date_day") String start_date_day, @RequestParam("end_date_day") String end_date_day) throws Exception {
		
		//http://localhost/plmetc/vault/getPublicationOfPartData
		//http://localhost/plmetc/vault/getPublicationOfPartData?
		//logger.info("VaultReportController.getExportDate = {}");

		ArrayList<HashMap<String, String>> result = PartCommonUtil.getPublicationOfPartData(partNo, specList, start_date_day, end_date_day);
		response.setHeader("Access-Control-Allow-Origin", "*");
		response.setHeader("Access-Control-Allow-Methods", "GET,POST,DELETE,PUT,OPTIONS");
		return result;
	}
	
	@RequestMapping(value = "/getPublicationOfPartData", method = RequestMethod.POST)
	@ResponseBody
	public ArrayList<HashMap<String, String>> getPartPublication(HttpServletResponse response, @RequestParam("partNo") String partNo, @RequestParam("specList") String specList, 
			@RequestParam("start_date_day") String start_date_day, @RequestParam("end_date_day") String end_date_day) throws Exception {
		
		//http://localhost/plmetc/vault/getPublicationOfPartData
		//http://localhost/plmetc/vault/getPublicationOfPartData?
		//logger.info("VaultReportController.getExportDate = {}");

		ArrayList<HashMap<String, String>> result = PartCommonUtil.getPublicationOfPartData(partNo, specList, start_date_day, end_date_day);
		response.setHeader("Access-Control-Allow-Origin", "*");
		response.setHeader("Access-Control-Allow-Methods", "GET,POST,DELETE,PUT,OPTIONS");
		return result;
	}
	
	
	/**
	 * @category 부품공용화_대시보드 
	 * @param response
	 * @param partNo
	 * @param partType
	 * @param startDate
	 * @param endDate
	 * @return 
	 * @throws Exception
	 */
	@RequestMapping(value = "/getPartPublicationSum", method = RequestMethod.POST)
	@ResponseBody
	public HashMap<String, Object> getPartPublicationSum(HttpServletResponse response, @RequestParam("partNo") String partNo
			, @RequestParam("partType") String partType, @RequestParam("startDate") String startDate, @RequestParam("endDate") String endDate) throws Exception {
		
		//http://localhost/vault/getPartPublicationSum
		//https://plmpro.hdel.co.kr/vault/getPartPublicationSum
		
		HashMap<String, Object> resultList = new HashMap<String, Object>();
		
		ArrayList<String> partNoList = new ArrayList<String>();
		if(partNo != null && !"".equals(partNo)) {
			String[] temp = partNo.split(",");
			for(int i=0; i < temp.length; i++) {
				partNoList.add(temp[i]);
			}
		}
		
		if(partNo != null && !"".equals(partNo)) {
			resultList = DashboardCommonUtil.searchPartPublication(partNoList, partType, "", startDate, endDate);
		}
		response.setHeader("Access-Control-Allow-Origin", "*");
		response.setHeader("Access-Control-Allow-Methods", "GET,POST,DELETE,PUT,OPTIONS");
		return resultList; 
	}
	
	
	   
	/**
	 * 도면정보(AutoCAD) 조회
	 * @param response
	 * @param dwgNo
	 * @return
	 * @throws Exception
	 */
	@RequestMapping(value = "/getAutoCADInfo", method = RequestMethod.GET)
	@ResponseBody 
	public HashMap<String, String> getAutoCADInfo(HttpServletResponse response, String dwgNo) throws Exception {
		
		//http://localhost/plmetc/vault/getPublicationOfPartData
		//https://plmpro.hdel.co.kr/plmetc/vault/getAutoCADInfo
		//http://localhost/plmetc/vault/getAutoCADInfo?dwgNo=10100185
		//System.out.println("getAutoCADInfo == " + dwgNo);
		
		HashMap<String, String> result = new HashMap<String, String>();
		 
		if(dwgNo != null && !"".equals(dwgNo)) {
			result =VaultCommonUtil.getAutoCADInfo(dwgNo);
		}
		
		response.setHeader("Access-Control-Allow-Origin", "*");
		response.setHeader("Access-Control-Allow-Methods", "GET,POST,DELETE,PUT,OPTIONS"); 
		return result; 
	}
	
	
	/**
	 * 설계복사화면에서 호기 입력 시 제품의 버전, 상태 조회
	 * @param response
	 * @param dwgNo
	 * @return
	 * @throws Exception
	 */
	@RequestMapping(value = "/findProductInfo", method = RequestMethod.GET)
	@ResponseBody 
	public HashMap<String, String> findProductInfo(HttpServletResponse response, String productNo) throws Exception {
		
		//http://localhost/plmetc/vault/findProductInfo?productNo=N26185L01
		//https://plmpro.hdel.co.kr/plmetc/vault/findProductInfo?productNo=N26185L01
		
		HashMap<String, String> result = new HashMap<String, String>();
		 
		if(productNo != null && !"".equals(productNo)) {
			result =VaultCommonUtil.findProductInfo(productNo);
		}
		
		response.setHeader("Access-Control-Allow-Origin", "*");
		response.setHeader("Access-Control-Allow-Methods", "GET,POST,DELETE,PUT,OPTIONS"); 
		return result; 
	}
	

	
	/**N26143L01
     * 설계복사 > 제품의 하위 BOM 중 수정된 자재만 조회
     * http://localhost/plmetc/vault/findModParts?productNo=N26143L01
     * @param productNo
     * @return
     * @throws Exception
     */
	@RequestMapping(value = "/findModParts", method = RequestMethod.GET)
	@ResponseBody 
    public ArrayList<PartInfoDTO> findModParts(String productNo) throws Exception {
		//https://plmpro.hdel.co.kr/plmetc/vault/findModParts?productNo=N26185L01
        ArrayList<PartInfoDTO> result = new ArrayList<PartInfoDTO>();
        //result = InventorCommonUtil.findModParts(productNo);
        result = VaultCommonUtil.findModParts(productNo);
        return result;
    }
	
	

	/**
	 * localhost/plmetc/vault/findSalesLogic
	 * 영업사양 정보 추출
	 * @param elvOid
	 * @return
	 * @throws Exception
	 */
	@RequestMapping(value = "/findSalesLogic", method = RequestMethod.GET)
	@ResponseBody 
    public ArrayList<elvInfoDTO> findSalesLogic(String proudtNo) throws Exception {
		

    	HashMap<String, String> dataMap = PartCommonUtil.getLatestElvInfoOid(proudtNo);
    	String version = dataMap.get("VERSION");
    	System.out.println("dataMap -" + dataMap);
    	
    	
    	JdbcTemplate plmJdbcTemplate = DBconnectionInfo.getInstance().getPLMJdbcTemplate();
  	    List<Map<String, Object>> ouid = plmJdbcTemplate.queryForList("SELECT CONCAT('elv_info$vf@', LOWER(DECTOHEX(vf$ouid))) OUID FROM ELV_INFO$VF WHERE MD$NUMBER = '" + proudtNo + "' AND VF$VERSION = '" + version + "'");
  	    System.out.println("ouid -" + ouid);
  		String elvOuid = (String) ouid.get(0).get("OUID");
  		DOS dos = (DOS)SvServer.getServiceInstance("DF30DOS1");
    	
		ArrayList<elvInfoDTO> result = new ArrayList<elvInfoDTO>();
		result = PartCommonUtil.getSalesData(elvOuid);
		
        
        return result;
    }
	
    
    @RequestMapping(value = "/pidExecute", method = RequestMethod.GET)
	@ResponseBody 
    public HashMap<String, String> pidExecute(String hogi, String PID, String testVersion, String isfloor, String floor, String type) throws Exception {
		
    	
    	HashMap<String, String> result = new HashMap<String, String>();
    	
    	
    	// testVersion : 공백이면 최신버전으로, "on"이면 test버전으로 수행
    	// isfloor : Y이면 층벼 PID이다.
    	// floor : 몇층인지 정보
    	
    	HashMap<String, String> dataMap = PartCommonUtil.getLatestElvInfoOid(hogi);
    	String version = dataMap.get("VERSION");
    	
    	JdbcTemplate plmJdbcTemplate = DBconnectionInfo.getInstance().getPLMJdbcTemplate();
	    List<Map<String, Object>> ouid = plmJdbcTemplate.queryForList("SELECT CONCAT('elv_info$vf@', LOWER(DECTOHEX(vf$ouid))) OUID FROM ELV_INFO$VF WHERE MD$NUMBER = '" + hogi + "' AND VF$VERSION = '" + version + "'");
		String iOuid = (String) ouid.get(0).get("OUID");
		DOS dos = (DOS)SvServer.getServiceInstance("DF30DOS1");

    	
    	
    	//String iOuid = request.getParameter("iOuid");
    	//String PID = request.getParameter("PID");
    	String beforePids = ""; //request.getParameter("beforePids");
    	String PARTOUID = ""; //request.getParameter("PARTOUID");
    	//String isfloor = ""; //request.getParameter("isfloor"); --- Y
    	//String floor = ""; //request.getParameter("floor"); --- 몇층인지 정보
    	String inputType = ""; //request.getParameter("inputType");
    	//String sampleData = request.getParameter("sampleData");
    	//String testVersion = ""; //StringUtil.NVL(request.getParameter("testVersion"),"off");
    	//String pickPart  = StringUtil.NVL(request.getParameter("pickPart"),"off");

    	
    	//PidService pidService = new PidService();
    	PidDesignService pidService = new PidDesignService();
    	
    	VariantMap logicMap = null; 

    	PID = (PID == null)? null : PID.trim(); 
    	floor = (floor == null)? null : floor.trim(); 
    	
    	
    	//public static final String LASTEST = "lastest";
    	//public static final String TEST = "test";
    	//public static final String FLOOREXPANDS = "Y";
    	
    	//logicMap = pidService.debugWithOuid(iOuid, "Y".equals(isfloor)? floor : null, PID, "on".equals(testVersion)? PidConsts.TEST : PidConsts.LASTEST, beforePids);
    	
    	if(type != null && !"".equals(type) && "SELECT".equals(type)) {
    		logicMap = pidService.debugWithOuid(iOuid, "Y".equals(isfloor)? floor : null, PID, testVersion, beforePids);
    	} else {
    		logicMap = pidService.debugWithOuid(iOuid, "Y".equals(isfloor)? floor : null, PID, "on".equals(testVersion)? PidConsts.TEST : PidConsts.LASTEST, beforePids);
    	}
    	
    	
    	
    	
    	
    	VariantMap debugData = (VariantMap) logicMap.get("debugData");

    	// JAVA로 실행한 결과는 데이터구조가 다르니 별도로 출력하고 끝낸다

    	HashMap localelvEnt = (HashMap) logicMap.get("elvEnt");
    	
    	// ouid로 실제 영업사양 값을 재 저장한다.
    	//DOS dos = (DOS) SvServer.getServiceInstance(ServiceID.DOS);
    	DOSChangeable elvData = null;
    	if (!iOuid.equals("") && !iOuid.equals("null") && iOuid != null){
    		elvData = (DOSChangeable) dos.get(iOuid);
    	}
    	
    	Variant variant = new Variant(localelvEnt, null);
    	
    	int maxSpecIdx = StringUtil.parseInt(debugData.get("maxSpecIdx"));
    	int maxResIdx  = StringUtil.parseInt(debugData.get("maxResIdx"));
    	ArrayList data = (ArrayList)debugData.get("data");
    	

    	for(int i=0; i<data.size(); i++)
			{
				HashMap row = (HashMap) data.get(i);
				ArrayList specList = (ArrayList) row.get("specList");
				ArrayList conList = (ArrayList) row.get("conList");
				ArrayList compareResultList = (ArrayList) row.get("compareResultList");

				ArrayList keyList = (ArrayList) row.get("keyList");
				ArrayList valList = (ArrayList) row.get("valList");
				String ADDR = StringUtil.NVL(row.get("ADDR"),"&nbsp;");
				String GOTO = StringUtil.NVL(row.get("GOTO"),"&nbsp;");
				String REMARKS = StringUtil.NVL(row.get("REMARKS"),"&nbsp;");
				
				VariantMap resultMap = (VariantMap) row.get("resultMap");
				String resultMapStr = StringUtil.NVL(resultMap,"&nbsp;").replaceAll("\n","<BR>");
				String rowTrue = StringUtil.NVL(row.get("rowTrue"),"");
				String isBlankLine = StringUtil.NVL(row.get("isBlankLine"),"");
				
				String tdClass_access = "";
				String tdClass_compareRow = "";
				
				if(rowTrue.equals(""))
					tdClass_access = "unreaded";
				else
				{
					tdClass_access = "readed";
					
					if(rowTrue.equals("true"))
						tdClass_compareRow = "rowTrue";
					else if(rowTrue.equals("false"))
						tdClass_compareRow = "rowFalse";
				}
				
				//sb.append("<tr class='"+tdClass_compareRow+" "+tdClass_access+"'>");
				//sb.append("<td nowrap class='leftSide "+tdClass_access+" "+tdClass_compareRow+"'>"+(i+1)+"</td>");
				//sb.append("<td nowrap class='addr "+tdClass_access+" "+tdClass_compareRow+"'>"+ADDR+"</td>");
				
				
				// SPEC, CON DATA
				for(int j=0; j<maxSpecIdx; j++)
				{
					
					String spec =  StringUtil.NVL(specList.get(j),"&nbsp;");
					String specVal =  StringUtil.NVL(localelvEnt.get(spec),"");

					
					String specTitle = "";
					String compareCell = "";
					
					if(!spec.equals(""))
						specTitle = StringUtil.getDosfldTitle(spec);
					
					String con = StringUtil.NVL(conList.get(j),"&nbsp;");
						 			
					if(compareResultList != null && compareResultList.size()>j)
					 compareCell = StringUtil.NVL(compareResultList.get(j),"");
					
					String tdClass_compareCell = "";
					if(!compareCell.equals(""))
					{
 					if(compareCell.equals("T"))
 						tdClass_compareCell = "cellTrue";
 					else if(compareCell.equals("F"))
 						tdClass_compareCell = "cellFalse";
					}
					

 				//sb.append("<td nowrap class='"+tdClass_compareCell+" "+tdClass_access+" "+tdClass_compareRow+"' title='"+specTitle+"'>"+spec+"</td>");
 				//sb.append("<td nowrap class='"+tdClass_compareCell+" "+tdClass_access+" "+tdClass_compareRow+"' title='"+specVal+"'>"+con+"</td>");
				}
				
				String output = "";
			 	ArrayList duplicationAvoid = new ArrayList();
			 	
				// KEY, VAL DATA
				for(int j=0; j<maxResIdx; j++)
				{
					
					String key = StringUtil.NVL(keyList.get(j),"&nbsp;");
					String val = StringUtil.NVL(valList.get(j),"&nbsp;").replaceAll("\n","<BR>");

 				//sb.append("<td nowrap class='key"+(j+1)+" "+tdClass_access+" "+tdClass_compareRow+"'>"+key+"</td>");
 				//sb.append("<td nowrap class='"+tdClass_access+" "+tdClass_compareRow+"'>"+val+"</td>");
 				
	 				if(key.equals("&nbsp;") == false && tdClass_compareRow.equals("rowTrue"))
	 				{
	 					
	 					if(output.equals("") == false)
	 						output += "<BR>";
	 					try
	 					{
	 						if(duplicationAvoid.contains(key) == false)
	 						{
	 							duplicationAvoid.add(key);
	 							if(key.equals("CALL"))
	 							{
	 								output += key+"("+val+") = "+resultMap.toString().replace("\n","<BR>");
	 							}
	 							else
	 							{
	 								output += key+" = "+StringUtil.NVL(resultMap.get(key),"");
	 								
	 								System.out.println(key + " = " + StringUtil.NVL(resultMap.get(key),"") );
	 								
	 								String outVal = StringUtil.NVL(resultMap.get(key),"");
	 								result.put(key.trim(), outVal.trim());
	 							}
	 							
	 							if(val.contains("[$"))
	 							{
	 								output += " ("+variant.convertVal(val, localelvEnt, elvData)+")";
	 							}
	 						}
	 					}
	 					catch(Exception e)
	 					{
	 						e.printStackTrace();
	 						output="에러!";
	 					}
	 				}
	 				
	 				
	 				if( !output.contains("OUTPUT") && !output.contains("CALL") ) {
						if(output != null && !"".equals(output)) {
							
							String[] str = output.split("=");
							
							String k = str[0];
							
							if(str[1] != null) {
								String v = str[1];
								//result.put(k.trim(), v.trim());
							    
							}
						}
					}
	 				
				} //end for
				
				//System.out.println("output == " + output);
			}
        
        return result;
    }
    
    
    
    //LineData
    @RequestMapping(value = "/pidExecute", method = RequestMethod.GET)
	@ResponseBody 
    public ArrayList pidExecuteLineData(String hogi, String PID, String testVersion, String isfloor, String floor, String type) throws Exception {
		
    	
    	// testVersion : 공백이면 최신버전으로, "on"이면 test버전으로 수행
    	// isfloor : Y이면 층벼 PID이다.
    	// floor : 몇층인지 정보
    	
    	HashMap<String, String> dataMap = PartCommonUtil.getLatestElvInfoOid(hogi);
    	String version = dataMap.get("VERSION");
    	
    	JdbcTemplate plmJdbcTemplate = DBconnectionInfo.getInstance().getPLMJdbcTemplate();
	    List<Map<String, Object>> ouid = plmJdbcTemplate.queryForList("SELECT CONCAT('elv_info$vf@', LOWER(DECTOHEX(vf$ouid))) OUID FROM ELV_INFO$VF WHERE MD$NUMBER = '" + hogi + "' AND VF$VERSION = '" + version + "'");
		String iOuid = (String) ouid.get(0).get("OUID");
		DOS dos = (DOS)SvServer.getServiceInstance("DF30DOS1");

    	
    	
    	//String iOuid = request.getParameter("iOuid");
    	//String PID = request.getParameter("PID");
    	String beforePids = ""; //request.getParameter("beforePids");
    	String PARTOUID = ""; //request.getParameter("PARTOUID");
    	//String isfloor = ""; //request.getParameter("isfloor"); --- Y
    	//String floor = ""; //request.getParameter("floor"); --- 몇층인지 정보
    	String inputType = ""; //request.getParameter("inputType");
    	
    	//PidService pidService = new PidService();
    	PidDesignService pidService = new PidDesignService();
    	
    	VariantMap logicMap = null; 

    	PID = (PID == null)? null : PID.trim(); 
    	floor = (floor == null)? null : floor.trim(); 
    	
    	
    	//public static final String LASTEST = "lastest";
    	//public static final String TEST = "test";
    	//public static final String FLOOREXPANDS = "Y";
    	
    	//logicMap = pidService.debugWithOuid(iOuid, "Y".equals(isfloor)? floor : null, PID, "on".equals(testVersion)? PidConsts.TEST : PidConsts.LASTEST, beforePids);
    	
    	if(type != null && !"".equals(type) && "SELECT".equals(type)) {
    		logicMap = pidService.debugWithOuid(iOuid, "Y".equals(isfloor)? floor : null, PID, testVersion, beforePids);
    	} else {
    		logicMap = pidService.debugWithOuid(iOuid, "Y".equals(isfloor)? floor : null, PID, "on".equals(testVersion)? PidConsts.TEST : PidConsts.LASTEST, beforePids);
    	}
    	
    	VariantMap debugData = (VariantMap) logicMap.get("debugData");

    	// JAVA로 실행한 결과는 데이터구조가 다르니 별도로 출력하고 끝낸다

    	HashMap localelvEnt = (HashMap) logicMap.get("elvEnt");
    	
    	// ouid로 실제 영업사양 값을 재 저장한다.
    	//DOS dos = (DOS) SvServer.getServiceInstance(ServiceID.DOS);
    	DOSChangeable elvData = null;
    	if (!iOuid.equals("") && !iOuid.equals("null") && iOuid != null){
    		elvData = (DOSChangeable) dos.get(iOuid);
    	}
    	
    	Variant variant = new Variant(localelvEnt, null);
    	
    	int maxSpecIdx = StringUtil.parseInt(debugData.get("maxSpecIdx"));
    	int maxResIdx  = StringUtil.parseInt(debugData.get("maxResIdx"));
    	ArrayList data = (ArrayList)debugData.get("data");
    	

        return data;
    }
    
    
    //http://localhost/plmetc/vault/getFloorInfo?prodNum=N26143L01
    @RequestMapping(value = "/getFloorInfo", method = RequestMethod.GET)
   	@ResponseBody 
    public ArrayList<HashMap<String, Object>> getFloorInfo(String prodNum) throws Exception {

    	//HashMap<String,Object> pickMap = new HashMap<String,Object>();
    	
    	
    	ArrayList<HashMap<String, Object>> result = new ArrayList<>();
    	
    	//층별 모든 자재 (key는 층, value는 hashmap으로 key는 pick, value는 자재번호)
    	LinkedHashMap<String, HashMap<String, String>> floorMap = new LinkedHashMap<String, HashMap<String, String>>();

    	LinkedHashMap<String, String> floorCompareMap = new LinkedHashMap<String, String>();
    	
    	//header이면서 현 제품에 속해있는 층별 PICK정보 (중복제거)
    	ArrayList<String> headerPickList = new ArrayList<String>();
    	
    	ArrayList<String> inputOptList = new ArrayList<String>();  //getInputBlockOptionList(request);
		//inputOptList.add("CRT");
		inputOptList.add("C");
		inputOptList.add("M");
		inputOptList.add("F");
		inputOptList.add("1");
		inputOptList.add("2");
		inputOptList.add("3");
		
    	
    	HashMap<String, String> dataMap = PartCommonUtil.getLatestElvInfoOid(prodNum);
		String elvVersion = dataMap.get("VERSION");
		
		JdbcTemplate plmJdbcTemplate = DBconnectionInfo.getInstance().getPLMJdbcTemplate();                                         
	    List<Map<String, Object>> ouidddList = plmJdbcTemplate.queryForList("SELECT CONCAT('elv_info$vf@', LOWER(DECTOHEX(vf$ouid))) OUID FROM ELV_INFO$VF WHERE MD$NUMBER = '" + prodNum + "' AND VF$VERSION = '" + elvVersion + "'");
	    
	    
	    if(ouidddList != null && ouidddList.size() > 0) {
	    	String elvOuid = (String) ouidddList.get(0).get("OUID");
			SubaeManagerPick manager = new SubaeManagerPick(elvOuid);
			
			result = manager.bomALLFloorCalculate(inputOptList, "2035570", headerPickList, floorMap);
			
			//System.out.println("pickMap == " + pickMap);
			//System.out.println("---------------------------------------------------");
			//System.out.println("floorMap == " + floorMap);
	    }
    	
    	
    	return result;
    }
}

```