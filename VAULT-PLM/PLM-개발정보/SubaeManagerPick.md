


```
package dyna.plmetc.subae.model;

import dyna.framework.service.DOS;
import dyna.framework.service.ServiceID;
import dyna.framework.service.dos.DOSChangeable;
import dyna.framework.starter.SvServer;
import dyna.plmetc.mybatis.PDMMyBatisConnectionFactory;
import dyna.plmetc.spring.service.part.PartService;
import dyna.plmetc.util.Calculator;
import dyna.plmetc.util.VariantUtil;
import dyna.plmetc.variant.Variant;
import dyna.plmetc.variant.VariantMap;
import dyna.plmetc.util.DBconnectionInfo;
import dyna.plmetc.util.StringUtil;
import dyna.util.Utils;
import java.math.BigDecimal;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.Date;
import java.util.HashMap;
import java.util.Iterator;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;
import java.util.stream.Collectors;

import org.apache.ibatis.session.SqlSession;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.web.context.ContextLoader;
import org.springframework.web.context.WebApplicationContext;


/**
 * @category
 * @author helco2035570
 *
 */
public class SubaeManagerPick {
	private static final Logger logger = LoggerFactory.getLogger(SubaeManagerPick.class);
	private DOS dos;
	private String elvOuid;

	private SubaeDaoImpl subaeDao;
	
	private ElvMaster elvMaster;
	private List<FloorMaster> floorMasterList;
	//private ProductInfo productInfo;
	private ProductInfoManager productInfo;
	
	private PartListSeeker seeker;
	
	private Map<String, String[]> tempVariableMap = null;
	private ArrayList<String> computedPartList;
	private List<Map<String, String>> tempVariableList;
	private Map<String, Map<String, String>> tempVariablePartMap;

	private Map<String, List<Map<String, String>>> variablePartMap4Simulate; // 시뮬레이션 전용 공사수량 Collection

	private ArrayList<HashMap<String, Object>> floorInfoList = new ArrayList<HashMap<String,Object>>();
	
	private SimpleDateFormat sdf = new SimpleDateFormat("yyyyMMddHHmmss");
	
	public SubaeManagerPick() throws Exception {
		this.dos = (DOS)SvServer.getServiceInstance(ServiceID.DOS);
		this.subaeDao = new SubaeDaoImpl();
	}
	
	public SubaeManagerPick(String elvOuid) throws Exception {
		this();
		this.elvOuid = elvOuid;
		checkElvOuid();
	}
	
	public Map getElvInfo() throws Exception{
		return subaeDao.getElvInfo(elvOuid);
	}
	
	private void checkElvOuid() throws Exception {
		if (elvOuid.startsWith(SubaeConstants.PREFIX_PRODUCT_OUID)) {
			HashMap filter = new HashMap();
			filter.put("version.condition.type", "wip");
			filter.put("list.mode", "association");
			filter.put("ouid@association.class", SubaeConstants.ELVANDPRODUCT_ASSO_OUID);
			ArrayList list = dos.listLinkTo(elvOuid, filter);
			if (Utils.isNullArrayList(list)) {
				throw new Exception("최신 버전 제품이 아니거나, 연결된 공사정보가 없습니다.<br>BOM 계산을 할 수 없습니다.");
			}
			elvOuid = (String)((ArrayList)list.get(0)).get(0);
		}
	}
	
	public String getElvStatus() throws Exception{
		return dos.getStatus(elvOuid);
	}
	
	public void checkElvVersionStatus() throws Exception {
		ArrayList ouidList = dos.getListVersionHistoryOuid(elvOuid);
		if (Utils.isNullArrayList(ouidList)) {
			throw new Exception("공사정보가 없습니다.");
		}
		
		if (ouidList.size() > 1) {
			String lastOuid = (String)ouidList.get(ouidList.size() - 1);
			if (!elvOuid.equals(lastOuid)) {
				throw new Exception("최신 공사정보에서만 BOM계산이 가능합니다.");
			}
		}
	}
	
	public HashMap<String, Object> bomCalculate(ArrayList<String> inputOptList, String userId, HashMap<String,String> separateInfo) throws Exception {
		//System.out.println("========= SubaeManagerPick.bomCalculate =============");
		//System.out.println("========= Check Elv Status =============");
		String status = dos.getStatus(elvOuid);
		//System.out.println("status = " + status);
		//System.out.println("elvOuid = " + elvOuid);
		
		if("RLS".equals(status)) {
			//throw new Exception("이미 승인된 버전입니다. WIP생성 후 BOM계산 해주세요.");
			System.out.println("searchProductOneLevel === 이미 승인된 버전입니다. WIP생성 후 BOM계산 해주세요." + status);
		}

		
		//1.영업사양 값 셋팅
		System.out.println("========= Create Elv & Floor Master =============");
		elvMaster = new ElvMaster(elvOuid, dos);
		elvMaster.setDataMap();
		
		
		//System.out.println("elvMaster.getDataMap() ---> " + elvMaster.getDataMap()); //영업사양 값들 들어있다
		
		
		
		//2. 층 정보 셋팅
		//elvMaster.addSubSpec(subaeDao.getSubSpecList(elvOuid));
		if (elvOuid.startsWith(SubaeConstants.PREFIX_ELVINFO_OUID)) {
			setFloorMasterList();
		}
		
		System.out.println("========= Make EL_P Spec =============");
		if (elvOuid.startsWith(SubaeConstants.PREFIX_ELVINFO_OUID)) {
			calculate_EL_P();
			floorMasterList.forEach(floorMaster -> {
				floorMaster.getDataMap().putAll(elvMaster.getDataMap());
			});
			
		}else if (elvOuid.startsWith(SubaeConstants.PREFIX_SVELVINFO_OUID)) {
			inputOptList.add("1");
			inputOptList.add("2");
			inputOptList.add("3");
			inputOptList.add("M");
			inputOptList.add("F");
			calculate_SVEL_P();
		}
		else if (elvOuid.startsWith(SubaeConstants.PREFIX_SHIPELVINFO_OUID)) {
			calculate_SHIPEL_P();
		}
		
		
		System.out.println("========= Ready Product =============");
		productInfo = new ProductInfoManager(elvMaster.getObjectInfo(), userId, dos);
		productInfo.readyProduct4Calculate();
		productInfo.compareBlockOption(inputOptList);
		
		System.out.println("productInfo.getBlockOptList4Calc() ==  " + productInfo.getBlockOptList4Calc());
		
		//if (Utils.isNullArrayList(productInfo.getBlockOptList4Calc())) return;
		
		
		
		System.out.println("========= Create and Ready PartListSeeker =============");
		this.tempVariableMap = new LinkedHashMap<String, String[]>();
		this.computedPartList = new ArrayList<String>();
		this.tempVariableList = new ArrayList<Map<String, String>>();
		this.tempVariablePartMap = new LinkedHashMap<String, Map<String, String>>();

		List<BlockInfo> blockList = null;
		List<BlockInfo> floorBlockList = null;

		if (elvOuid.startsWith(SubaeConstants.PREFIX_ELVINFO_OUID))
			blockList = subaeDao.getBlockList(productInfo.getBlockOptList4Calc(), false);
		else if (elvOuid.startsWith(SubaeConstants.PREFIX_SVELVINFO_OUID))
			blockList = subaeDao.getBlockList(productInfo.getBlockOptList4Calc(), false);
		else
			blockList = subaeDao.getShipBlockList(productInfo.getBlockOptList4Calc());

		if(floorMasterList != null)
			floorBlockList = subaeDao.getBlockList(productInfo.getBlockOptList4Calc(), true);

		
		//System.out.println("blockList ==  " + blockList);
		//System.out.println("floorBlockList ==  " + floorBlockList);
		
		
		HashMap<String, Object> pickMap = new HashMap<String, Object>();
		
		pickMap = pickAndCalculatePid(blockList, floorBlockList, separateInfo);
		
		/*
		 * Iterator<String> itr = pickMap.keySet().iterator(); while(itr.hasNext()) {
		 * String key = itr.next(); //System.out.println(key + " >> " +
		 * pickMap.get(key)); }
		 */
		

		System.out.println("========= Make EBOM Structure and Insert VariablePart Start =============");
		//makeEBomStructure(userId);

		//DOSChangeable elvInfo = elvMaster.getObjectInfo();
		//elvInfo.put("ebom_manager",userName);
		//if (elvInfo.isChanged()) dos.set(elvInfo);
		
		//releaseElvInfo();
		//productInfo.updateCalcOptList();
		
		return pickMap;
	}
	
	
	//층별자재 전용 추출
	public HashMap<String, Object> bomFloorCalculate(ArrayList<String> inputOptList, 
			String userId, ArrayList<String> headerPickList, LinkedHashMap<String, HashMap<String, String>> floorMap 
			 ) throws Exception {
		//System.out.println("========= SubaeManagerPick.bomFloorCalculate =============");
		String status = dos.getStatus(elvOuid);
		//System.out.println("elvOuid = " + elvOuid);
		
		if("RLS".equals(status)) {
			//throw new Exception("이미 승인된 버전입니다. WIP생성 후 BOM계산 해주세요.");
			System.out.println("searchProductOneLevel === 이미 승인된 버전입니다. WIP생성 후 BOM계산 해주세요." + status);
		}

		
		//1.영업사양 값 셋팅
		System.out.println("========= Create Elv & Floor Master =============");
		elvMaster = new ElvMaster(elvOuid, dos);
		elvMaster.setDataMap();
		
		
		//System.out.println("elvMaster.getDataMap() ---> " + elvMaster.getDataMap()); //영업사양 값들 들어있다
		
		
		
		//2. 층 정보 셋팅
		//elvMaster.addSubSpec(subaeDao.getSubSpecList(elvOuid));
		if (elvOuid.startsWith(SubaeConstants.PREFIX_ELVINFO_OUID)) {
			setFloorMasterList();
		}
		
		System.out.println("========= Make EL_P Spec =============");
		if (elvOuid.startsWith(SubaeConstants.PREFIX_ELVINFO_OUID)) {
			calculate_EL_P();
			floorMasterList.forEach(floorMaster -> {
				floorMaster.getDataMap().putAll(elvMaster.getDataMap());
			});
			
		}else if (elvOuid.startsWith(SubaeConstants.PREFIX_SVELVINFO_OUID)) {
			inputOptList.add("1");
			inputOptList.add("2");
			inputOptList.add("3");
			inputOptList.add("M");
			inputOptList.add("F");
			calculate_SVEL_P();
		}
		else if (elvOuid.startsWith(SubaeConstants.PREFIX_SHIPELVINFO_OUID)) {
			calculate_SHIPEL_P();
		}
		
		
		System.out.println("========= Ready Product =============");
		productInfo = new ProductInfoManager(elvMaster.getObjectInfo(), userId, dos);
		productInfo.readyProduct4Calculate();
		productInfo.compareBlockOption(inputOptList);
		
		System.out.println("productInfo.getBlockOptList4Calc() ==  " + productInfo.getBlockOptList4Calc());
		
		//if (Utils.isNullArrayList(productInfo.getBlockOptList4Calc())) return;
		
		
		
		System.out.println("========= Create and Ready PartListSeeker =============");
		this.tempVariableMap = new LinkedHashMap<String, String[]>();
		this.computedPartList = new ArrayList<String>();
		this.tempVariableList = new ArrayList<Map<String, String>>();
		this.tempVariablePartMap = new LinkedHashMap<String, Map<String, String>>();

		List<BlockInfo> blockList = null;
		List<BlockInfo> floorBlockList = null;

		if (elvOuid.startsWith(SubaeConstants.PREFIX_ELVINFO_OUID))
			blockList = subaeDao.getBlockList(productInfo.getBlockOptList4Calc(), false);
		else if (elvOuid.startsWith(SubaeConstants.PREFIX_SVELVINFO_OUID))
			blockList = subaeDao.getBlockList(productInfo.getBlockOptList4Calc(), false);
		else
			blockList = subaeDao.getShipBlockList(productInfo.getBlockOptList4Calc());

		if(floorMasterList != null)
			floorBlockList = subaeDao.getBlockList(productInfo.getBlockOptList4Calc(), true);

		
		
		ArrayList<HashMap<String, Object>> rr = new ArrayList<HashMap<String,Object>>();
		
		HashMap<String, Object> pickMap = new HashMap<String, Object>();
		
		
		//현 제품에 속해있는 층별 PICK정보 (중복제거)
		ArrayList<String> allPickInFO = new ArrayList<String>();
		
		
		pickMap = pickAndCalculatePidFloor(blockList, floorBlockList, headerPickList, floorMap);
		
		return pickMap;
	}
	
	
	//층별 전체 정보 가져오기
	public ArrayList<HashMap<String, Object>> bomALLFloorCalculate(ArrayList<String> inputOptList, 
			String userId, ArrayList<String> headerPickList, LinkedHashMap<String, HashMap<String, String>> floorMap 
			 ) throws Exception {
		String status = dos.getStatus(elvOuid);
		
		if("RLS".equals(status)) {
			//throw new Exception("이미 승인된 버전입니다. WIP생성 후 BOM계산 해주세요.");
			System.out.println("searchProductOneLevel === 이미 승인된 버전입니다. WIP생성 후 BOM계산 해주세요." + status);
		}

		
		//1.영업사양 값 셋팅
		System.out.println("========= Create Elv & Floor Master =============");
		elvMaster = new ElvMaster(elvOuid, dos);
		elvMaster.setDataMap();
		
		//System.out.println("elvMaster.getDataMap() ---> " + elvMaster.getDataMap()); //영업사양 값들 들어있다
		
		//2. 층 정보 셋팅
		//elvMaster.addSubSpec(subaeDao.getSubSpecList(elvOuid));
		if (elvOuid.startsWith(SubaeConstants.PREFIX_ELVINFO_OUID)) {
			setFloorMasterList();
		}
		
		System.out.println("========= Make EL_P Spec =============");
		if (elvOuid.startsWith(SubaeConstants.PREFIX_ELVINFO_OUID)) {
			calculate_EL_P();
			floorMasterList.forEach(floorMaster -> {
				floorMaster.getDataMap().putAll(elvMaster.getDataMap());
			});
			
		}else if (elvOuid.startsWith(SubaeConstants.PREFIX_SVELVINFO_OUID)) {
			inputOptList.add("1");
			inputOptList.add("2");
			inputOptList.add("3");
			inputOptList.add("M");
			inputOptList.add("F");
			calculate_SVEL_P();
		}
		else if (elvOuid.startsWith(SubaeConstants.PREFIX_SHIPELVINFO_OUID)) {
			calculate_SHIPEL_P();
		}
		
		
		System.out.println("========= Ready Product =============");
		productInfo = new ProductInfoManager(elvMaster.getObjectInfo(), userId, dos);
		productInfo.readyProduct4Calculate();
		productInfo.compareBlockOption(inputOptList);
		
		//System.out.println("productInfo.getBlockOptList4Calc() ==  " + productInfo.getBlockOptList4Calc());
		
		//if (Utils.isNullArrayList(productInfo.getBlockOptList4Calc())) return;
		
		
		
		System.out.println("========= Create and Ready PartListSeeker =============");
		this.tempVariableMap = new LinkedHashMap<String, String[]>();
		this.computedPartList = new ArrayList<String>();
		this.tempVariableList = new ArrayList<Map<String, String>>();
		this.tempVariablePartMap = new LinkedHashMap<String, Map<String, String>>();

		List<BlockInfo> blockList = null;
		List<BlockInfo> floorBlockList = null;

		if (elvOuid.startsWith(SubaeConstants.PREFIX_ELVINFO_OUID))
			blockList = subaeDao.getBlockList(productInfo.getBlockOptList4Calc(), false);
		else if (elvOuid.startsWith(SubaeConstants.PREFIX_SVELVINFO_OUID))
			blockList = subaeDao.getBlockList(productInfo.getBlockOptList4Calc(), false);
		else
			blockList = subaeDao.getShipBlockList(productInfo.getBlockOptList4Calc());

		if(floorMasterList != null)
			floorBlockList = subaeDao.getBlockList(productInfo.getBlockOptList4Calc(), true);

		
		
		ArrayList<HashMap<String, Object>> rr = new ArrayList<HashMap<String,Object>>();
		
		HashMap<String, Object> pickMap = new HashMap<String, Object>();
		
		
		//현 제품에 속해있는 층별 PICK정보 (중복제거)
		//ArrayList<String> allPickInFO = new ArrayList<String>();
		
		
		//pickMap = pickAndCalculatePidFloor(blockList, floorBlockList, headerPickList, floorMap);
		//pickMap = pickAndCalculateALLPidFloor(blockList, floorBlockList, headerPickList, floorMap);
		
		return floorInfoList;
	}
	

	private HashMap<String, Object> pickAndCalculatePid(List<BlockInfo> blockList, List<BlockInfo> floorBlockList
									, HashMap<String,String> separateInfo) throws Exception {
		// Common Block Pick & PID Calculate
		
		HashMap<String, Object> pickMap = new HashMap<String, Object>();
		//HashMap<String, String> separateInfo = new HashMap<String, String>();
		
		{
			HashMap dataMap = elvMaster.getDataMap();
			VariableAction variableAction = new VariableAction(dataMap, VariantUtil.convertToMapList(floorMasterList));
			for (BlockInfo blockInfo : blockList) {
				findBom(dataMap, variableAction, blockInfo, pickMap, separateInfo);
			}
		}

		// Floor Block Pick & PID Calculate
		if (floorMasterList != null) {
			//System.out.println("floorMasterList = " + floorMasterList);
			for (FloorMaster floorMaster : floorMasterList) {
				//System.out.println("floorMaster ----------- " + floorMaster.getDataMap());
				HashMap dataMap = floorMaster.getDataMap();
				
				String floorName = (String) dataMap.get("FLOOR_NAME");
				//System.out.println("floorName >>>>>>>>>>>>>>>>>>>>>>>>>> " + floorName);
				VariableAction variableAction = new VariableAction(dataMap, VariantUtil.convertToMapList(floorMasterList));
				for (BlockInfo blockInfo : floorBlockList) {
					//System.out.println("floorMasterList BLOCKLIST = " + blockInfo);
					findBom(dataMap, variableAction, blockInfo, pickMap, separateInfo);
				}
			}
		}

		tempVariableList.addAll(tempVariablePartMap.entrySet().stream().map(Entry::getValue).collect(Collectors.toList()));
		//pickMap.put("separateInfo", separateInfo); // pick,수량 분리 객체
		
		return pickMap;
	}
	
	//BOM 1레벨 PICK, 수량 분리 추출 프로그램
	private HashMap<String, Object> pickSeparateCalculatePid(List<BlockInfo> blockList, List<BlockInfo> floorBlockList) throws Exception {
		// Common Block Pick & PID Calculate
		
		HashMap<String, Object> pickMap = new HashMap<String, Object>();
		HashMap<String, String> separateInfo = new HashMap<String, String>();
		
		ArrayList<HashMap<String, Object>> rr = new ArrayList<HashMap<String,Object>>();
		ArrayList<HashMap<String, Object>> aa = new ArrayList<HashMap<String,Object>>();
		{
			HashMap dataMap = elvMaster.getDataMap();
			VariableAction variableAction = new VariableAction(dataMap, VariantUtil.convertToMapList(floorMasterList));
			for (BlockInfo blockInfo : blockList) {
				findBom(dataMap, variableAction, blockInfo, pickMap, separateInfo);
			}
		}

		// Floor Block Pick & PID Calculate
		if (floorMasterList != null) {
			//System.out.println("floorMasterList = " + floorMasterList);
			for (FloorMaster floorMaster : floorMasterList) {
				//System.out.println("floorMaster ----------- " + floorMaster.getDataMap());
				HashMap dataMap = floorMaster.getDataMap();
				
				String floorName = (String) dataMap.get("FLOOR_NAME");
				System.out.println("floorName >>>>>>>>>>>>>>>>>>>>>>>>>> " + floorName);
				VariableAction variableAction = new VariableAction(dataMap, VariantUtil.convertToMapList(floorMasterList));
				for (BlockInfo blockInfo : floorBlockList) {
					//System.out.println("floorMasterList BLOCKLIST = " + blockInfo);
					findBom(dataMap, variableAction, blockInfo, pickMap, separateInfo);
				}
			}
		}

		tempVariableList.addAll(tempVariablePartMap.entrySet().stream().map(Entry::getValue).collect(Collectors.toList()));
		
		return pickMap;
	}
	
	
	//층별 자재 추출을 위한 함수
	private HashMap<String, Object> pickAndCalculatePidFloor(List<BlockInfo> blockList, List<BlockInfo> floorBlockList
								, ArrayList<String> headerPickList, LinkedHashMap<String, HashMap<String, String>> floorMap) throws Exception {
		// Common Block Pick & PID Calculate
		
		HashMap<String, Object> pickMap = new HashMap<String, Object>();
		
		//ArrayList<HashMap<String, Object>> rr = new ArrayList<HashMap<String,Object>>();
		//ArrayList<HashMap<String, Object>> aa = new ArrayList<HashMap<String,Object>>();
		
		
		
		// Floor Block Pick & PID Calculate
		if (floorMasterList != null) {
			//System.out.println("floorMasterList = " + floorMasterList);
			for (FloorMaster floorMaster : floorMasterList) {
				HashMap dataMap = floorMaster.getDataMap();
				
				String floorName = (String) dataMap.get("FLOOR_NAME");
				System.out.println("floorName >>>>>>>>>>>>>>>>>>>>>>>>>> " + floorName);
				VariableAction variableAction = new VariableAction(dataMap, VariantUtil.convertToMapList(floorMasterList));
				
				//층 맵
				
				HashMap<String, String> partMap = new HashMap<String, String>();
				
				for (BlockInfo blockInfo : floorBlockList) {
					//findBom(dataMap, variableAction, blockInfo, pickMap);
					findFloorBom(dataMap, variableAction, blockInfo, pickMap, floorName, partMap, headerPickList);
				}
				
				floorMap.put(floorName, partMap);
			}
			//System.out.println("floorMap = " + floorMap);
		}

		tempVariableList.addAll(tempVariablePartMap.entrySet().stream().map(Entry::getValue).collect(Collectors.toList()));
		
		return pickMap;
	}
	
	//전체층 전체 정보
	private HashMap<String, Object> pickAndCalculateALLPidFloor(List<BlockInfo> blockList, List<BlockInfo> floorBlockList
			, ArrayList<String> headerPickList, LinkedHashMap<String, HashMap<String, String>> floorMap) throws Exception {
		// Common Block Pick & PID Calculate
		
		HashMap<String, Object> pickMap = new HashMap<String, Object>();
		
		
		// Floor Block Pick & PID Calculate
		if (floorMasterList != null) {
		//System.out.println("floorMasterList = " + floorMasterList);
			for (FloorMaster floorMaster : floorMasterList) {
				HashMap dataMap = floorMaster.getDataMap();
				
				String floorName = (String) dataMap.get("FLOOR_NAME");
				System.out.println("floorName >>>>>>>>>>>>>>>>>>>>>>>>>> " + floorName);
				VariableAction variableAction = new VariableAction(dataMap, VariantUtil.convertToMapList(floorMasterList));
			
				//층 맵
			
				HashMap<String, String> partMap = new HashMap<String, String>();
				
				for (BlockInfo blockInfo : floorBlockList) {
					//findFloorBom(dataMap, variableAction, blockInfo, pickMap, floorName, partMap, headerPickList);
					findALLFloorBom(dataMap, variableAction, blockInfo, pickMap, floorName, partMap, headerPickList);
				}
				
				floorMap.put(floorName, partMap);
			}
		}
		
		
		tempVariableList.addAll(tempVariablePartMap.entrySet().stream().map(Entry::getValue).collect(Collectors.toList()));
		
		return pickMap;
	}
	
	
	
	private HashMap<String, Object> findFloorBom(HashMap dataMap, VariableAction vAction, BlockInfo blockInfo, 
			HashMap<String, Object> oMap, String floorName, 
			HashMap<String, String> partMap, ArrayList<String> headerPickList)
			throws Exception {
		
		//HashMap<String, Object> oMap = new HashMap<String, Object>();
		//ArrayList<HashMap<String, Object>> returnMap = new ArrayList<HashMap<String,Object>>();
		
		List<PickInfo> pickList = blockInfo.getPickList();
		
		ArrayList<String> keyBlockNo = new ArrayList<String>();
		keyBlockNo.add("C263B01");
		keyBlockNo.add("D161A");
		keyBlockNo.add("C263A01");
		keyBlockNo.add("C265A01");
		keyBlockNo.add("C264B09");
		keyBlockNo.add("C263B10");
		keyBlockNo.add("C263A04");
		keyBlockNo.add("C263A10");
		keyBlockNo.add("C263B04");
		keyBlockNo.add("C361A");
		keyBlockNo.add("C371A01");
		keyBlockNo.add("C262A01");
		keyBlockNo.add("C371A");
		keyBlockNo.add("D375A");
		
		
		HashMap<String, String> targetMap = new HashMap<String, String>();
		
		
		for (PickInfo pickInfo : pickList) {
			String pick = pickInfo.getPick();
			String qty = pickInfo.getQty();
			String cmt = pickInfo.getCmt();
			String color = pickInfo.getColor();
			
			//System.out.println("pick >>> " + pick);
			if(pick != null) {
				pick = pick.trim();
			}
			
			Map picked = pickPart(dataMap, blockInfo.getOuid(), pick);
			if (picked != null) {
				String partOuid = (String) picked.get("OUID");
				boolean hasChild = !BigDecimal.ZERO.equals(picked.get("HASCHILD"));

				String partNo = (String) picked.get("PARTNO");
				//String pickValue = (String) picked.get("PICK");
				
				//System.out.println(partNo + " >>> " );
				if( !"".equals(pick) ) {
					//System.out.println(pick + " >>> " + picked);
					
					// 층별 정보 출력
					//System.out.println(pick + " >>> " + picked.get("PARTNO") + ", " + picked.get("VER") + ", " + picked.get("PART_STATUS") + " , " + picked.get("G_L_CODE") + " , " + picked.get("B_NO") + " , " + picked.get("SPEC"));
					//System.out.println(picked.get("PARTNO") + " >>> " + pick + ", " + qty + " , " + picked.get("VER") + ", " + " , " + picked.get("SPEC") ); 
					
					String pickBlockNo = String.valueOf(picked.get("B_NO")); // BlockNo
					String pickPartNo = String.valueOf(picked.get("PARTNO")); // PartNo
					
					partMap.put(pickBlockNo, pickPartNo);
					
					
					if(!headerPickList.contains(pickBlockNo)) {
						headerPickList.add(pickBlockNo);
					}
					
					
					if(keyBlockNo.contains(pickBlockNo)) {
						targetMap.put(pickBlockNo, pickPartNo);
					}
					
					//System.out.println(picked.get("PARTNO") + " >>> " + pick + ", qty : " + qty);
					
					
					if(oMap.containsKey(partNo)) {
						String tempValue = (String) oMap.get(partNo);
						
						if( !tempValue.contains(pick) ) {
							tempValue += "&" + pick;
							oMap.put(partNo, tempValue);
						}
						
					} else {
						oMap.put(partNo, pick);
					}
				}
				
				
				// Lv1 PID Calculate
				{
					picked.put("PICK", pick);
					
					//수량계산
					String[] variable = vAction.get1LevelVariable(picked, qty, cmt, color);
					setTempVariableMap(partOuid, variable);
					
					//수량과 픽번호
					//System.out.println(picked.get("PARTNO") + " >>> " + pick + ", qty : " + variable[1]);
				}

				// Child PID Calculate
				/*
				if(hasChild){
					boolean isCalculated = true;
					if (Utils.isNullArrayList(computedPartList) || !computedPartList.contains(partOuid)) {
						computedPartList.add(partOuid);
						isCalculated = false;
					}

					ArrayList<Map<String, String>> variableList = new ArrayList<>();
					List<Map> listPartOfPart = subaeDao.getListPartOfPart(partOuid);
					for (Map partOfPart : listPartOfPart) {
						String assoOuid = String.valueOf(partOfPart.get("SF$OUID"));
						String end2PartOuid = String.valueOf(partOfPart.get("END2_HEXOUID"));

						String child_qty = (String) partOfPart.get("QTY");
						String child_cmt = (String) partOfPart.get("CMT");
						String child_color = (String) partOfPart.get("COLOR");

						if (StringUtil.isPidPattern(child_qty) || StringUtil.isPidPattern(child_cmt)
								|| StringUtil.isPidPattern(child_color)) {

							HashMap<String, String> childPartInfo = generatePartInfo(partOfPart);

							HashMap<String, String> variable = vAction.getOtherLevelVariable(assoOuid, childPartInfo,
									child_qty, child_cmt, child_color, isCalculated);
							variableList.add(variable);
						}
					}
					
					if(variablePartMap4Simulate != null && variableList.size() > 0){
						variablePartMap4Simulate.put(partOuid, variableList);
					}
					setTempVariableList(partOuid, variableList);
				}
				*/
			}
		} // end for
		
		return oMap;
	}
	
	
	//층별 전체 속성값
	private HashMap<String, Object> findALLFloorBom(HashMap dataMap, VariableAction vAction, BlockInfo blockInfo, 
			HashMap<String, Object> oMap, String floorName, 
			HashMap<String, String> partMap, ArrayList<String> headerPickList)
			throws Exception {
		
		//HashMap<String, Object> oMap = new HashMap<String, Object>();
		//ArrayList<HashMap<String, Object>> returnMap = new ArrayList<HashMap<String,Object>>();
		
		List<PickInfo> pickList = blockInfo.getPickList();
	
		HashMap<String, String> targetMap = new HashMap<String, String>();
		
		
		for (PickInfo pickInfo : pickList) {
			String pick = pickInfo.getPick();
			String qty = pickInfo.getQty();
			String cmt = pickInfo.getCmt();
			String color = pickInfo.getColor();
			
			//System.out.println("pick >>> " + pick);
			if(pick != null) {
				pick = pick.trim();
			}
			
			Map picked = pickPart(dataMap, blockInfo.getOuid(), pick);
			if (picked != null) {
				String partOuid = (String) picked.get("OUID");
				boolean hasChild = !BigDecimal.ZERO.equals(picked.get("HASCHILD"));

				String partNo = (String) picked.get("PARTNO");
				if( !"".equals(pick) ) {
					
					String pickBlockNo = String.valueOf(picked.get("B_NO")); // BlockNo
					String pickPartNo = String.valueOf(picked.get("PARTNO")); // PartNo
					
					partMap.put(pickBlockNo, pickPartNo);
					if(!headerPickList.contains(pickBlockNo)) {
						headerPickList.add(pickBlockNo);
					}
					
					targetMap.put(pickBlockNo, pickPartNo);
					
					//System.out.println(picked.get("PARTNO") + " >>> " + pick + ", qty : " + qty);
					if(oMap.containsKey(partNo)) {
						String tempValue = (String) oMap.get(partNo);
						
						if( !tempValue.contains(pick) ) {
							tempValue += "&" + pick;
							oMap.put(partNo, tempValue);
						}
						
					} else {
						oMap.put(partNo, pick);
					}
				}
				
				
				// Lv1 PID Calculate
				{
					picked.put("PICK", pick);
					
					//수량계산
					String[] variable = vAction.get1LevelVariable(picked, qty, cmt, color);
					setTempVariableMap(partOuid, variable);
					
					//수량과 픽번호
					//System.out.println(picked.get("PARTNO") + " >>> " + pick + ", qty : " + variable[1]);
				}

			}
		} // end for
		
		return oMap;
	}

	private HashMap<String, Object> findBom(HashMap dataMap, VariableAction vAction, BlockInfo blockInfo, 
			HashMap<String, Object> oMap, HashMap<String, String> separateInfo)
			throws Exception {
		
		List<PickInfo> pickList = blockInfo.getPickList();
		
		
		for (PickInfo pickInfo : pickList) {
			String pick = pickInfo.getPick();
			String qty = pickInfo.getQty();
			String cmt = pickInfo.getCmt();
			String color = pickInfo.getColor();
			
			if(pick != null) {
				pick = pick.trim();
			}
			
			Map picked = pickPart(dataMap, blockInfo.getOuid(), pick);
			if (picked != null) {
				String partOuid = (String) picked.get("OUID");
				boolean hasChild = !BigDecimal.ZERO.equals(picked.get("HASCHILD"));

				String partNo = (String) picked.get("PARTNO");
				//String pickValue = (String) picked.get("PICK");
				
				//System.out.println(partNo + " >>> " );
				if( !"".equals(pick) ) {
					//System.out.println(pick + " >>> " + picked);
					
					// 층별 정보 출력
					//System.out.println(pick + " >>> " + picked.get("PARTNO") + ", " + picked.get("VER") + ", " + picked.get("PART_STATUS") + " , " + picked.get("G_L_CODE") + " , " + picked.get("B_NO") + " , " + picked.get("SPEC"));
					
					//System.out.println(picked.get("PARTNO") + " >>> " + pick + ", " + qty + " , " + picked.get("VER") + ", " + " , " + picked.get("SPEC") ); 
					//System.out.println(picked.get("PARTNO") + " >>> " + pick + ", qty : " + qty);
					
					//하나의 품번에 PICK번호 여러개일 경우 합치기
					if(oMap.containsKey(partNo)) {
						String tempValue = (String) oMap.get(partNo);
						
						if( !tempValue.contains(pick) ) {
							tempValue += "&" + pick;
							oMap.put(partNo, tempValue);
						}
					} else {
						oMap.put(partNo, pick);
					}
				}
				
				
				// Lv1 PID Calculate
				{
					picked.put("PICK", pick);
					
					//수량계산
					//영업사양 정보->주석,수량
					//0: 주석
					//1: 수량
					//2: 
					//3: 수배된 자재번호(품번)
					//4: 블럭번호
					String[] variable = vAction.get1LevelVariable(picked, qty, cmt, color);
					setTempVariableMap(partOuid, variable);
					
					
					String keyInfo = picked.get("PARTNO") + "-" + pick;
					String valInfo = variable[1];
					
					separateInfo.put(keyInfo, valInfo);
					
					/*
					 * if(separateInfo.containsKey(keyInfo)) { separateInfo.put(keyInfo, valInfo); }
					 * else { separateInfo.put(keyInfo, valInfo); }
					 */
					
					//System.out.println("EACH --- :: " + picked.get("PARTNO") + " >>> " + pick + ", qty : " + variable[1]);
				}

				// Child PID Calculate
				/*
				if(hasChild){
					boolean isCalculated = true;
					if (Utils.isNullArrayList(computedPartList) || !computedPartList.contains(partOuid)) {
						computedPartList.add(partOuid);
						isCalculated = false;
					}

					ArrayList<Map<String, String>> variableList = new ArrayList<>();
					List<Map> listPartOfPart = subaeDao.getListPartOfPart(partOuid);
					for (Map partOfPart : listPartOfPart) {
						String assoOuid = String.valueOf(partOfPart.get("SF$OUID"));
						String end2PartOuid = String.valueOf(partOfPart.get("END2_HEXOUID"));

						String child_qty = (String) partOfPart.get("QTY");
						String child_cmt = (String) partOfPart.get("CMT");
						String child_color = (String) partOfPart.get("COLOR");

						if (StringUtil.isPidPattern(child_qty) || StringUtil.isPidPattern(child_cmt)
								|| StringUtil.isPidPattern(child_color)) {

							HashMap<String, String> childPartInfo = generatePartInfo(partOfPart);

							HashMap<String, String> variable = vAction.getOtherLevelVariable(assoOuid, childPartInfo,
									child_qty, child_cmt, child_color, isCalculated);
							variableList.add(variable);
						}
					}
					
					if(variablePartMap4Simulate != null && variableList.size() > 0){
						variablePartMap4Simulate.put(partOuid, variableList);
					}
					setTempVariableList(partOuid, variableList);
				}
				*/
			}
		} // end for
		
		return oMap;
	}

	
	private HashMap<String, String> generatePartInfo(Map partOfPart) {
		HashMap<String, String> childPartInfo = new HashMap<>();
		childPartInfo.put("PARTNO", String.valueOf(partOfPart.get("PARTNO")));
		childPartInfo.put("G_L_CODE", String.valueOf(partOfPart.get("G_L_CODE")));
		childPartInfo.put("SPEC", String.valueOf(partOfPart.get("SPEC")));
		childPartInfo.put("PART_SIZE", String.valueOf(partOfPart.get("PART_SIZE")));
		childPartInfo.put("B_NO", String.valueOf(partOfPart.get("B_NO")));
		return childPartInfo;
	}

	public List<FloorMaster> getFloorMasterList() {
		return floorMasterList;
	}

	public Map<String, String[]> bomSimulate(List<String> blockList) throws Exception {
		List<Map<String, Object>> res = new ArrayList<Map<String, Object>>();
		
		
		WebApplicationContext context = ContextLoader.getCurrentWebApplicationContext();
		PartService partService = context.getBean("partService", PartService.class);

		List<BlockInfo> blockInfoList = null;
		List<BlockInfo> floorBlockInfoList = null;
		HashMap<String,String> separateInfo = new HashMap<String,String>();
		
		elvMaster = new ElvMaster(elvOuid, dos);
		elvMaster.setDataMap();
		//elvMaster.addSubSpec(subaeDao.getSubSpecList(elvOuid));
		if (elvOuid.startsWith(SubaeConstants.PREFIX_ELVINFO_OUID)) {
			setFloorMasterList();
			blockInfoList = subaeDao.findBLocksByNo(blockList, false);
			floorBlockInfoList = subaeDao.findBLocksByNo(blockList, true);
		}else{
			blockInfoList = subaeDao.findBLocksByNo(blockList);
		}

		if (elvOuid.startsWith(SubaeConstants.PREFIX_ELVINFO_OUID)) {
			calculate_EL_P();
		}
		else if (elvOuid.startsWith(SubaeConstants.PREFIX_SHIPELVINFO_OUID)) {
			calculate_SHIPEL_P();
		}else if (elvOuid.startsWith(SubaeConstants.PREFIX_SVELVINFO_OUID)) {
			calculate_SVEL_P();
		}

//		Create and Ready PartListSeeker
		this.tempVariableMap = new LinkedHashMap<>();
		this.computedPartList = new ArrayList<>();
		this.tempVariableList = new ArrayList<>();
		this.tempVariablePartMap = new LinkedHashMap<>();
		this.variablePartMap4Simulate = new LinkedHashMap<>();

		pickAndCalculatePid(blockInfoList, floorBlockInfoList, separateInfo);

		return tempVariableMap;
	}
	
	private void calculate_EL_P() throws Exception {
		long start = System.nanoTime();
		System.out.println("==== cal ELV EL_P ====");
		EL_P el_p = new EL_P(subaeDao);
		el_p.set_EL_P_pidList();
		el_p.make_EL_P_Data(elvMaster.getDataMap(), floorMasterList, false);
		
		if (Utils.isNullList(this.floorMasterList)) {
			return;
		}
		
		System.out.println("==== cal Floors EL_P ====");
		for (FloorMaster floorMaster : floorMasterList) {
			//System.out.println("-- floor : " + floorMaster.getObjectInfo().get("md$description"));
			el_p.make_EL_P_Data(floorMaster.getDataMap(), floorMasterList, true);
		}
		
		for (FloorMaster floorMaster : floorMasterList) {
			//System.out.println("-- floor : " + floorMaster.getObjectInfo().get("md$description"));
			el_p.make_EL_P_Data(floorMaster.getDataMap(), floorMasterList, true);
			
			//FLOOR_NAME
			HashMap dataMap = floorMaster.getDataMap();
			String floorName = (String) dataMap.get("FLOOR_NAME");
			
			
			
			//System.out.println("floorMaster.getDataMap() == " + floorMaster.getDataMap());
			HashMap temp = calc_EL_P(floorMaster.getDataMap(), floorMasterList, true);
			
			HashMap<String, Object> floorTemp = new HashMap<String, Object>();
			floorTemp.put(floorName, temp);
			
			floorInfoList.add(floorTemp);
		}
		
		
		
		long end = System.nanoTime();
		System.out.println("calculate_EL_P() end : " + Calculator.parse("[$ round(" + (end - start) / 1000.0 / 1000 + ",3)$]") + "ms");
	}
	
	private HashMap calc_EL_P(HashMap dataInfoMap, List<FloorMaster> floorMasterList, boolean isFloorSpec) throws Exception {	
		Variant variant = new Variant(dataInfoMap, VariantUtil.convertToMapList(floorMasterList));
		HashMap<String, String> resultMap = new HashMap<String, String>();
		String ouid = (String)dataInfoMap.get("ouid");
		
		SqlSession sessionOpen = PDMMyBatisConnectionFactory.sqlSessionOpen();
		SubaeDao subaeDao = sessionOpen.getMapper(SubaeDao.class);
		
		//subaeDao.getEL_PList(isFloorSpec ? "Y" : "N");
		List<Map> pidList = subaeDao.getEL_PList("Y");
		
		
		//List<Map> pidList = isFloorSpec ? floor_pidList : elv_pidList;		
		System.out.println("pidList = " + pidList);
		
		if (pidList == null || pidList.size() ==0) {
			return resultMap;
		}
		
		for(Map pidMap : pidList) {
			String pid = (String)pidMap.get("PID");
			String method = (String)pidMap.get("METHOD");

			VariantMap localMap = null;

			try {
				localMap = variant.calcVariantPID(pid, null);
			} catch (Exception e) {
				System.err.println(pid + "(" + method + ") : " + e.getMessage());
			}

			if (localMap != null) {
				Iterator<String> it = localMap.getOUTPUTMap().keySet().iterator();

				while (it.hasNext()) {
					String key = (String) it.next();
					String value = StringUtil.NVL(localMap.get(key), "");
					resultMap.put(key, value);
				}
			}
		}
		
		return resultMap;
	}
	
	
	private void calculate_SHIPEL_P() throws Exception {
		long start = System.nanoTime();
		System.out.println("==== cal SHIP_ELV EL_P ====");
		EL_P el_p = new EL_P(subaeDao);
		el_p.set_SHIPEL_P_pidList();
		el_p.make_EL_P_Data(elvMaster.getDataMap(), null, false);
		
		if (Utils.isNullList(this.floorMasterList)) {
			return;
		}
		long end = System.nanoTime();
		//System.out.println("calculate_EL_P() end : " + Calculator.parse("[$ round(" + (end - start) / 1000.0 / 1000 + ",3)$]") + "ms");
	}

	
	private void calculate_SVEL_P() throws Exception {
		long start = System.nanoTime();
		System.out.println("==== cal SV_ELV EL_P ====");
		EL_P el_p = new EL_P(subaeDao);
		el_p.set_SVEL_P_pidList();
		el_p.make_EL_P_Data(elvMaster.getDataMap(), null, false);
		
		if (Utils.isNullList(this.floorMasterList)) {
			return;
		}
		long end = System.nanoTime();
		System.out.println("calculate_EL_P() end : " + Calculator.parse("[$ round(" + (end - start) / 1000.0 / 1000 + ",3)$]") + "ms");
	}
	private void setFloorMasterList() throws Exception {
		ArrayList<String> floorOuidList = getFloorOuidList();
		if (Utils.isNullArrayList(floorOuidList)) {
			return;
		}

		floorMasterList = new ArrayList<FloorMaster>();
		for (String ouid : floorOuidList) {
			FloorMaster floor = new FloorMaster(ouid, dos);
			floor.setDataMap();
			floor.addElvObjectData(elvMaster.getDataMap());
			floorMasterList.add(floor);
		}
		
		floorMasterList = floorMasterList.stream().sorted((p1,p2) ->{
			return Integer.compare(StringUtil.parseInt(p1.getDataMap().get("md$index")), StringUtil.parseInt(p2.getDataMap().get("md$index")));
		}).collect(Collectors.toList());

	}
	
	private void setFloorMasterList(ArrayList<DOSChangeable> floorInfoList) throws Exception {
		if (Utils.isNullArrayList(floorInfoList)) return;

		floorMasterList = new ArrayList<FloorMaster>();
		for (DOSChangeable floorInfo : floorInfoList) {
			FloorMaster floor = new FloorMaster(floorInfo, dos);
			floor.setDataMap();
			floor.addElvObjectData(elvMaster.getDataMap());
			floorMasterList.add(floor);
		}
		
		floorMasterList = floorMasterList.stream().sorted((p1,p2) ->{
			return Integer.compare(StringUtil.parseInt(p1.getDataMap().get("md$index")), StringUtil.parseInt(p2.getDataMap().get("md$index")));
		}).collect(Collectors.toList());
	}
	
	private ArrayList<String> getFloorOuidList() {
		ArrayList<String> floorOuidList = new ArrayList<String>();
		HashMap filter = new HashMap();
		filter.put("version.condition.type", "wip");
		filter.put("list.mode", "association");
		filter.put("navigableonly", false);
		filter.put("ouid@association.class", SubaeConstants.ELVANDFLOOR_ASSO_OUID);
		
		ArrayList list = dos.listLinkFrom(elvOuid, filter);
		if (Utils.isNullArrayList(list))
			return null;
		
		for (int i =0; i<list.size(); i++) {
			String ouid = (String)((ArrayList)list.get(i)).get(0);
			
			//System.out.println("")
			
			floorOuidList.add(ouid);
		}
		
		return floorOuidList;
	}
	
	private void setTempVariableList(String partOuid, ArrayList<Map<String, String>> variableList) {
		for(Map<String, String> variable : variableList) {
			if(tempVariablePartMap.containsKey(variable.get("assoOuid"))){
				Map<String, String> tempVar = tempVariablePartMap.get(variable.get("assoOuid"));
				String tempCmt = tempVar.get("cmt");
				String tempColor = tempVar.get("color");
				if (Utils.isNullString(tempCmt)) {
					tempVar.put("cmt", variable.get("cmt"));
				} else {
					if (!StringUtil.isDupCmt(tempCmt, variable.get("cmt"))) {
						tempVar.put("cmt", tempCmt + variable.get("cmt"));
					}
				}
				if (Utils.isNullString(tempColor)) {
					tempVar.put("color", variable.get("color"));
				} else {
					if (!StringUtil.isDupCmt(tempColor, variable.get("color"))) {
						tempVar.put("color", tempColor +  variable.get("color"));
					}
				}

			}else{
				tempVariablePartMap.put(variable.get("assoOuid"), variable);
			}
		}
	}

	private void setTempVariableMap(String partOuid, String[] variable) {
		if (tempVariableMap.get(partOuid) == null) {
			tempVariableMap.put(partOuid, variable);
		} else {
			String[] tmpVar = tempVariableMap.get(partOuid);
			String cmts = variable[0];
			if (!Utils.isNullString(cmts)) {
				if (Utils.isNullString(tmpVar[0])) {
					tmpVar[0] = cmts;
				} else {
					String[] cmt = cmts.split("\n");
					tmpVar[0] += tmpVar[0].endsWith("\n") ? "" : "\n";
					for (int i=0; i<cmt.length; i++) {
						tmpVar[0] += StringUtil.addLine(tmpVar[0], cmt[i]);
					}
				}
			}
			
			BigDecimal tmpQty;
			BigDecimal qty;
			
			try {
				qty = new BigDecimal(variable[1]);
			} catch (NumberFormatException e) {
				qty = BigDecimal.ZERO;
				System.err.println("qty numberformatException "+partOuid+", qty:"+variable[1]);
			}
			
			try {
				tmpQty = new BigDecimal(tmpVar[1]);
			} catch (NumberFormatException e) {
				tmpQty = BigDecimal.ZERO;
				System.err.println("qty numberformatException "+partOuid+", tmpQty:"+tmpVar[1]);
			}
			
			
			// 수량이 계산되지 않았을 경우는 문자열 표시
			if (qty != BigDecimal.ZERO && tmpQty != BigDecimal.ZERO) {
				tmpVar[1] = qty.add(tmpQty).stripTrailingZeros().toPlainString();
			}
		}
	}

	private void makeEBomStructure(String userId) throws Exception {
		String date = sdf.format(new Date());
				
		// BOM 연결 (partofpart)
		link1LevelPart(userId, date);

		String prodOuid = productInfo.getProdcutOuid();
		long lProdOuid = Utils.getRealLongOuid(prodOuid);
		//이미 수배된 VARIABLEPART 자재 들고오기
		List<String> variablePartListCheck = subaeDao.getVariablePartList(lProdOuid);
		
		int count = 0;
		for (Map<String, String> variable : tempVariableList) {
			String assoOuid = variable.get("assoOuid");
			String cmt = variable.get("cmt");
			String qty = variable.get("qty");
			String color = variable.get("color");

			//수배된 파트가 있을경우 수배대상에서 제외
			if(variablePartListCheck.contains(assoOuid))
				continue;

			// 2level variable part insert
			//System.out.println("2Level part insert variablepart table");
			// 공사 주석, 수량, 도장이 있을 경우만 입력
			if (!Utils.isNullString(cmt) || !Utils.isNullString(qty) | !Utils.isNullString(color)) {
				subaeDao.insertVariablePartRow(lProdOuid, Long.parseLong(assoOuid), qty, cmt, color);
				count++;
			}
		}
		System.out.println("2Level part size : " + tempVariableList.size());
		System.out.println("2Level part insert variablepart_new table : " + count + " rows");
	}
	
	private void link1LevelPart(String userId, String date) throws Exception {
		System.out.println("1Level part link and insert variablepart table");
		String prodOuid = productInfo.getProdcutOuid();
		long lProdOuid = Utils.getRealLongOuid(prodOuid);
		//이미 수배된 1레벨 PARTOFEBOM 자재 들고오기
		List<String> partListCheck = subaeDao.getPartList(lProdOuid);
		// 1level 링크 전 파트번호로 정렬
		List<String> keyList = new ArrayList<String>();
		keyList.addAll(tempVariableMap.keySet());
		Collections.sort(keyList, new Comparator<String>() {
			public int compare(String k1, String k2) {
				String v1 = (tempVariableMap.get(k1)[4] != null && tempVariableMap.get(k1)[4].length() > 1)
						? tempVariableMap.get(k1)[4].substring(1)
						: "";
				String v2 = (tempVariableMap.get(k2)[4] != null && tempVariableMap.get(k2)[4].length() > 1)
						? tempVariableMap.get(k2)[4].substring(1)
						: "";
		        return ((Comparable<String>) v1).compareTo(v2);
		    }
		});

		
		int seq = subaeDao.getMaxSeq(lProdOuid);
		Iterator<String> itr = keyList.iterator();
		while(itr.hasNext()) {
			seq += 10;
			String partOuid = itr.next();
			String cmt = tempVariableMap.get(partOuid)[0];
			String qty = tempVariableMap.get(partOuid)[1];
			if (Utils.isNullString(qty)) {
				qty = "0";
			}
			String color = tempVariableMap.get(partOuid)[2];
			//수배된 파트가 있을경우 수배대상에서 제외	
			String partDecOuid = StringUtil.hexTodeci(partOuid.substring(14));
			if(partListCheck.contains(partDecOuid))
				continue;
			
			long lPartOuid = Utils.getRealLongOuid(partOuid);
			String div = subaeDao.getPartDiv(lPartOuid);
			String mBom = "";
			if (!Utils.isNullString(div) && (div.equals(SubaeConstants.CODE_VALUE_INNER) || div.equals(SubaeConstants.CODE_VALUE_OUTER)  || div.equals(SubaeConstants.CODE_VALUE_INNER_F))) {
				mBom = "T";
			}
			
			subaeDao.insert1LevelPartOfEBom(lProdOuid, lPartOuid, String.valueOf(seq), qty, cmt, color, userId, mBom, date);
		}
	}
	

	/** 타 프로그램에서 ElvMaster & floor master 활용할 때 사용 
	 * @throws Exception */
	public Map getElvInfoAndFloorInfoWithELP() throws Exception {
		Map result = new HashMap();
		
		System.out.println("========= Create Elv & Floor Master =============");
		elvMaster = new ElvMaster(elvOuid, dos);
		elvMaster.setDataMap();
		setFloorMasterList();
		
		// EL_P 연산 취소
//		System.out.println("========= Make EL_P Spec =============");
//		calculate_EL_P();
		
		result.put("elvMaster", elvMaster);
		result.put("floorMasterList", floorMasterList);
		
		return result;
	}
	
	private void releaseElvInfo() {
		dos.setStatus(elvOuid, "RLS");
	}

	private Map pickPart(HashMap dataInfoMap, String blockOuid, String pickField) {
		String glCode = (String)dataInfoMap.get(pickField);
		
		if(glCode == null || "".equals(glCode))
			return null;
		
//		List<Map> pickedList = searchPart(glCode, blockOuid);
		
		JdbcTemplate plmJdbcTemplate = DBconnectionInfo.getInstance().getPLMJdbcTemplate();
		
		String sql = null;
		
		if(glCode.length() == 11) {
			sql = " SELECT 'normalpart$vf@' || lower(dectohex(VF$OUID)) OUID, VF$VERSION VER, COD(PART_STATUS) PART_STATUS "
					+ " ,MD$NUMBER PARTNO, G_L_CODE, SPEC, PART_SIZE, (SELECT MD$NUMBER FROM BLOCKNO$SF WHERE SF$OUID=GETID(BLOCKNO)) AS B_NO, COD(ORIGIN_DIV) ORIGIN_DIV, COD(SPT) SPT,"
					+ " (SELECT COUNT(1) FROM PARTOFPART$AC WHERE AS$END1=A.VF$OUID AND ROWNUM=1) HASCHILD "
					+ " FROM NORMALPART$VF A, NORMALPART$ID B WHERE A.VF$OUID=B.ID$LAST "
					+ " AND MD$NUMBER LIKE ?||'_' AND BLOCKNO = ? ";
		}
		else{
			sql = " SELECT 'normalpart$vf@' || lower(dectohex(VF$OUID)) OUID, VF$VERSION VER, COD(PART_STATUS) PART_STATUS "
					+ " ,MD$NUMBER PARTNO, G_L_CODE, SPEC, PART_SIZE, (SELECT MD$NUMBER FROM BLOCKNO$SF WHERE SF$OUID=GETID(BLOCKNO)) AS B_NO, COD(ORIGIN_DIV) ORIGIN_DIV, COD(SPT) SPT "
					+ ", (SELECT COUNT(1) FROM PARTOFPART$AC WHERE AS$END1=A.VF$OUID AND ROWNUM=1) HASCHILD"
					+ " FROM NORMALPART$VF A, NORMALPART$ID B WHERE A.VF$OUID=B.ID$LAST "
					+ " AND MD$NUMBER =? AND BLOCKNO = ? ";
		}
		
		List<Map<String, Object>> pickedList = plmJdbcTemplate.queryForList(sql, new Object[] {glCode, blockOuid} );
		
		
		
		if (Utils.isNullList(pickedList)) return null;
		
		Map map = pickedList.get(0);
		String version = (String)map.get("VER");
		String part_status = StringUtil.NVL(map.get("PART_STATUS"), "");
		if (version.equals("wip")) return null;
		
		if (!part_status.equals("Active")) return null;
				
		String partOuid = (String)map.get("OUID");

		return map;
	}
	
	private Map pickCostPart(HashMap dataInfoMap, String blockOuid, String pickField) {
		String glCode = (String)dataInfoMap.get(pickField);
		
		if(glCode == null || "".equals(glCode))
			return null;
		
//		List<Map> pickedList = searchPart(glCode, blockOuid);
		
		JdbcTemplate plmJdbcTemplate = DBconnectionInfo.getInstance().getPLMJdbcTemplate();
		
		String sql = null;
		
		if(glCode.length() == 11) {
			sql = " SELECT 'normalpart$vf@' || lower(dectohex(VF$OUID)) OUID, VF$VERSION VER, COD(PART_STATUS) PART_STATUS "
					+ " ,MD$NUMBER PARTNO, G_L_CODE, SPEC, PART_SIZE, (SELECT MD$NUMBER FROM BLOCKNO$SF WHERE SF$OUID=GETID(BLOCKNO)) AS B_NO, COD(ORIGIN_DIV) ORIGIN_DIV, COD(SPT) SPT "
					+ " FROM NORMALPART$VF A, NORMALPART$ID B WHERE A.VF$OUID=B.ID$LAST "
					+ " AND MD$NUMBER LIKE ?||'_' AND ( BLOCKNO = ?  or (SELECT cod(block_opt) FROM BLOCKNO$SF WHERE SF$OUID=GETID(?))='X')";
		}
		else{
			sql = " SELECT 'normalpart$vf@' || lower(dectohex(VF$OUID)) OUID, VF$VERSION VER, COD(PART_STATUS) PART_STATUS "
					+ " ,MD$NUMBER PARTNO, G_L_CODE, SPEC, PART_SIZE, (SELECT MD$NUMBER FROM BLOCKNO$SF WHERE SF$OUID=GETID(BLOCKNO)) AS B_NO, COD(ORIGIN_DIV) ORIGIN_DIV, COD(SPT) SPT "
					+ " FROM NORMALPART$VF A, NORMALPART$ID B WHERE A.VF$OUID=B.ID$LAST "
					+ " AND MD$NUMBER =?          AND ( BLOCKNO = ?  or (SELECT cod(block_opt) FROM BLOCKNO$SF WHERE SF$OUID=GETID(?))='X')";
		}
		
		List<Map<String, Object>> pickedList = plmJdbcTemplate.queryForList(sql, new Object[] {glCode, blockOuid,blockOuid} );
		
		
		
		if (Utils.isNullList(pickedList)) return null;
		
		Map map = pickedList.get(0);
		String version = (String)map.get("VER");
		String part_status = StringUtil.NVL(map.get("PART_STATUS"), "");
		if (version.equals("wip")) return null;
		
		//if (!part_status.equals("Active")) return null;
				
		String partOuid = (String)map.get("OUID");

		return map;
	}

	public Map<String, List<Map<String, String>>> getVariablePartMap4Simulate() {
		return variablePartMap4Simulate;
	}
}
```