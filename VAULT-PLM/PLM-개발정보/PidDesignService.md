---
tags:
---



---


```
package dyna.plmetc.spring.service.pid;

import dyna.plmetc.floor.FloorConst;
import dyna.plmetc.plm_sd_soap.WebSrvHandler;
import dyna.plmetc.spring.service.common.RestConsts;
import dyna.plmetc.subae.model.ElvMaster;
import dyna.plmetc.subae.model.FloorMaster;
import dyna.plmetc.subae.model.SubaeManager;
import dyna.plmetc.util.VariantUtil;
import dyna.plmetc.variant.PidNotFoundException;
import dyna.plmetc.variant.Variant;
import dyna.plmetc.variant.VariantMap;
import dyna.plmetc.variant.dto.Pid;
import java.io.IOException;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;
import org.codehaus.jackson.JsonParseException;
import org.codehaus.jackson.map.JsonMappingException;
import org.codehaus.jackson.map.ObjectMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Service;
import org.springframework.util.StringUtils;

@Service
public class PidDesignService {
	@Autowired
	@Qualifier("plmJdbcTemplate")
	private JdbcTemplate plmJdbcTemplate;
	
	public VariantMap calculateWithMap(Map<String, Object> salesInfo, String pid, String floorExpands) throws Exception {
		Variant variant = initVaraintWithMap(salesInfo, floorExpands);
		
		return variant.calcVariantPID(pid, null);
	}
	
	public VariantMap calculateWithString(String specData, String specDataType, String floorExpands, String pid)
			throws Exception {
		

		Variant variant = initVariantWithString(specData, specDataType, floorExpands);

		return variant.calcVariantPID(pid, null);
	}

	public VariantMap calculateWithOuid(String iOuid, String floor, String pid) throws Exception {

		Variant variant = initVariantWithOuid(iOuid, floor);

		return variant.calcVariantPID(pid, null);
	}

	public VariantMap debugWithString(String specData, String specDataType, String floorExpands, String pid, String version,
			String beforePids) throws Exception {
		Variant variant = initVariantWithString(specData, specDataType, floorExpands);
		return debugPid(variant, pid, version, beforePids);
	}

	public VariantMap debugWithOuid(String iOuid, String floor, String pid, String version, String beforePids) throws Exception {
		Variant variant = initVariantWithOuid(iOuid, floor);
		return debugPid(variant, pid, version, beforePids);
	}

	private VariantMap debugPid(Variant variant, String pid, String version, String beforePids) throws Exception {
		if(StringUtils.hasText(beforePids)){
			for(String beforePID : beforePids.split("\n")){
				if(StringUtils.hasText(beforePID)){
					try {
						VariantMap result = variant.calcVariantPID(beforePID.trim(), null);
						variant.appendToElvEnt(result);
					}catch(PidNotFoundException ignored){}
				}
			}
		}

		/*if (PidConsts.TEST.equals(version))
			return variant.calcVariantPID(pid, Variant.TESTversion, null, true);
		else
			return variant.calcVariantPID(pid, null, null, 1, true);
			*/
		
		if (PidConsts.TEST.equals(version)) {
			//System.out.println("----- TESTTTTTT ");
			return variant.calcVariantPID(pid, Variant.TESTversion, null, true);
		} else if(PidConsts.LASTEST.equals(version)) {
			//System.out.println("----- LATEST ");
			return variant.calcVariantPID(pid, null, null, 1, true);
		} else {
			//System.out.println("----- SELECTTTTTTTTTTTTTTTT ");
			//calcVariantPID(String PID, int version, Map partInfo, boolean debug) throws Exception
			return variant.calcVariantPID(pid, Integer.parseInt(version), null, true);
		}
	}
	

	private Variant initVaraintWithMap(Map<String, Object> salesInfo, String floorExpands) throws Exception {
		HashMap<String, String> specMap = new HashMap<String, String>();
		
		Iterator<Entry<String, Object>> iterator = salesInfo.entrySet().iterator();
		while(iterator.hasNext()) {
			Entry<String, Object> next = iterator.next();
			if(next.getValue() != null)
				specMap.put(next.getKey(), next.getValue().toString());
		}
		
		List<FloorMaster> floorMasterList = null;

		floorMasterList = specData2FloorMaster(floorExpands, specMap);

		Variant variant = new Variant(specMap, VariantUtil.convertToMapList(floorMasterList));
		return variant;
	}


	private Variant initVariantWithString(String specData, String specDataType, String floorExpands)
			throws IOException, JsonParseException, JsonMappingException, Exception {
		HashMap<String, String> specMap = specData2Map(specData, specDataType);

		// put md$number when execute with sample data
		if(!specMap.containsKey("md$number")){
			String hogi = specMap.get("HOGI");
			String qtnum = specMap.get("QTNUM"); // 견적번호
			String qtser = specMap.get("QTSER"); // 차수
			String qthogi = specMap.get("QTHOGI"); // 호기번호
			specMap.put("md$number", (StringUtils.hasText(hogi)) ? hogi : String.format("%s_%s_%s", qtnum, qtser, qthogi));
		}

		List<FloorMaster> floorMasterList = null;

		floorMasterList = specData2FloorMaster(floorExpands, specMap);

		Variant variant = new Variant(specMap, VariantUtil.convertToMapList(floorMasterList));
		return variant;
	}

	public Variant initVariantWithOuid(String iOuid, String floor) throws Exception {
		SubaeManager subaeManager = new SubaeManager(iOuid);
		Map iInfoMasterMap = subaeManager.getElvInfoAndFloorInfoWithELP();
		ElvMaster elvMaster = (ElvMaster) iInfoMasterMap.get("elvMaster");
		List<FloorMaster> floorMasterList = (List<FloorMaster>) iInfoMasterMap.get("floorMasterList");
		HashMap elvEnt = null;
		if (floor != null && !"".equals(floor) && floorMasterList != null) {
			for (FloorMaster floorMaster : floorMasterList) {
				System.out.println(".getDataMap() : " + floorMaster.getDataMap());
				if (floor.equals(floorMaster.getDataMap().get (FloorConst.FIELD_NAME_FLOOR_NAME).toString())) {
					elvEnt = floorMaster.getDataMap();
					break;
				}
			}
		} else {
			elvEnt = elvMaster.getDataMap();
		}
		Variant variant = new Variant(elvEnt, VariantUtil.convertToMapList(floorMasterList));
		return variant;
	}

	private List<FloorMaster> specData2FloorMaster(String floorExpands, HashMap<String, String> specMap)
			throws Exception {
		if (PidConsts.FLOOREXPANDS.equals(floorExpands)) {
			WebSrvHandler handler = new WebSrvHandler();
			handler.expandFloor(specMap);
			return handler.getFloorMasterList();
		} else
			return null;
	}

	private HashMap<String, String> specData2Map(String specData, String specDataType)
			throws IOException, JsonParseException, JsonMappingException {
		HashMap<String, String> resultMap = null;

		if (RestConsts.JSON.equals(specDataType)) {
			ObjectMapper mapper = new ObjectMapper();
			Map inputMap = mapper.readValue(specData, Map.class);
			resultMap = new HashMap<String, String>(inputMap);
		} else if (RestConsts.LIST.equals(specDataType)) {
			resultMap = new HashMap<String, String>();
			String[] split = specData.split("\n");
			for (String line : split) {
				String[] keyValue = line.trim().split("=");
				if (keyValue.length >= 2)
					resultMap.put(keyValue[0], keyValue[1]);
				else
					resultMap.put(keyValue[0], "");
			}
		}
		return resultMap;
	}

	public List<Pid> findByPid(String pid) {
		pid = pid != null ? pid.trim().replace("*","%") : "";

		List<Pid> result = plmJdbcTemplate.query(
				"SELECT A.* FROM VARIANT_H A, VARIANT_ID B WHERE A.PID = B.PID AND A.HOUID = B.LAST_HOUID AND A.PID LIKE ? ORDER BY A.PID",
				new Object[]{pid}, Pid.getRowMapper());

		return result;
	}
}

```