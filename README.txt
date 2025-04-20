package com.allstate.claims.claimstatusapi.service;

import com.allstate.claims.claimstatusapi.converter.CodeDecodeHelper;
import com.allstate.claims.claimstatusapi.converter.LiabilityDetailModelBuilder;
import com.allstate.claims.claimstatusapi.entity.LiabilityData;
import com.allstate.claims.claimstatusapi.model.AffectedParticipantDetailModel;
import com.allstate.claims.claimstatusapi.model.LiabilityDetailModel;
import com.allstate.claims.claimstatusapi.model.LiabilityParticipantDetailModel;
import com.allstate.claims.claimstatusapi.repositories.LiabilityRepository;
import com.allstate.claims.claimstatusapi.util.Helper;
import com.allstate.claims.claimstatusapi.util.LiabilityServiceHelper;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

import static com.allstate.claims.claimstatusapi.constants.Constants.Contributory_Negligence;
import static com.allstate.claims.claimstatusapi.constants.Constants.EMPTY_STRING;
import static com.allstate.claims.claimstatusapi.constants.Constants.FINAL;
import static com.allstate.claims.claimstatusapi.constants.Constants.INITIAL;
import static com.allstate.claims.claimstatusapi.constants.Constants.Mod_Comparative_49;
import static com.allstate.claims.claimstatusapi.constants.Constants.Mod_Comparative_50;
import static com.allstate.claims.claimstatusapi.constants.Constants.Mod_Comparative_51;
import static com.allstate.claims.claimstatusapi.constants.Constants.Other;
import static com.allstate.claims.claimstatusapi.constants.Constants.PARTICIPANT_ROLE_CATEGORY;
import static com.allstate.claims.claimstatusapi.constants.Constants.Pure_Comparative;
import static com.allstate.claims.claimstatusapi.constants.Constants.Slight_comparative;
import static java.util.Collections.emptyList;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertNotNull;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.eq;
import static org.mockito.Mockito.lenient;
import static org.mockito.Mockito.times;
import static org.mockito.Mockito.verify;
import static org.mockito.Mockito.when;

@ExtendWith(MockitoExtension.class)
public class LiabilityServiceTest {

    @InjectMocks
    LiabilityService service;

    @Mock
    LiabilityRepository liabilityRepository;
    @Mock
    LiabilityServiceHelper liabilityServiceHelper;
    @Mock
    CodeDecodeHelper codeDecodeHelper;
    @Mock
    LiabilityDetailModelBuilder liabilityDetailModelBuilder;
    @Mock
    Helper helper;

    public static final String CLAIM_ID = "123";
    public static final String NEGLigence_RULE = "TestRule";
    public static final String OWNR = "OWNR";
    public static final String PIN = "PIN";
    public static final String CMT = "CMT";
    private List<String> primaryParticipantList;
    private List<String> affectedParticipantList;
    private List<String> primaryParticipantCMTList;
    private List<String> affectedParticipantCMTList;

    @BeforeEach
    public void setUp() {
        primaryParticipantList = Arrays.asList(PIN, OWNR);
        affectedParticipantList = Arrays.asList(CMT, OWNR);
        primaryParticipantCMTList = Arrays.asList(CMT, OWNR);
        affectedParticipantCMTList = Arrays.asList(PIN, OWNR);

        lenient().when(codeDecodeHelper.getCodeDecodeShortDesc(eq(PARTICIPANT_ROLE_CATEGORY), any()))
                .thenReturn("Decoded Role");
        lenient().when(liabilityServiceHelper.joinNonNullDistinctList(any(), any()))
                .thenReturn(Arrays.asList("Role1"));
        lenient().when(liabilityServiceHelper.joinNonNullDistinctStrings(any(), any(), any()))
                .thenReturn("StringValue");
        lenient().when(service.getFinancialLiability(any(), any(), any(), any(), any(), any()))
               .thenReturn("CalculatedLiability");
        lenient().when(service.isLiabilityModified(any(), any(), any(), any()))
               .thenReturn(false);
    }

    @Test
    void getLiabilityDetails_emptyList() {
        when(liabilityRepository.getAllByClaimId(CLAIM_ID)).thenReturn(emptyList());
        LiabilityDetailModel expectedModel = new LiabilityDetailModel();
        when(liabilityDetailModelBuilder.build(any(), any(), any(), any())).thenReturn(expectedModel);

        LiabilityDetailModel result = service.getLiabilityDetails(CLAIM_ID, NEGLigence_RULE);

        assertNotNull(result);
        verify(liabilityRepository, times(1)).getAllByClaimId(CLAIM_ID);
        verify(liabilityServiceHelper, times(1)).getStatus(emptyList());
        verify(liabilityServiceHelper, times(1)).getStatusCode(any());
        verify(liabilityDetailModelBuilder, times(1)).build(any(), any(), eq(emptyList()), eq(NEGLigence_RULE));
    }

    @Test
    void getLiabilityDetails_singleEntry() {
        List<LiabilityData> liabilityDataList = Collections.singletonList(new LiabilityData());
        LiabilityParticipantDetailModel mappedParticipant = createMockParticipant("P1", "Affected1", "Key1", "50", "50", "P_ROLE1", "A_ROLE1");

        when(liabilityRepository.getAllByClaimId(CLAIM_ID)).thenReturn(liabilityDataList);
        when(liabilityServiceHelper.getStatus(liabilityDataList)).thenReturn("Status");
        when(liabilityServiceHelper.getStatusCode("Status")).thenReturn(INITIAL);
        when(liabilityServiceHelper.mapToObject(any(LiabilityData.class))).thenReturn(mappedParticipant);
        when(liabilityServiceHelper.getGroupKey(mappedParticipant)).thenReturn("Key1");

        when(liabilityServiceHelper.joinNonNullDistinctList(any(), any())).thenReturn(Arrays.asList("P_ROLE1"), Arrays.asList("A_ROLE1")); // Mock roles for this specific case
        when(codeDecodeHelper.getCodeDecodeShortDesc(eq(PARTICIPANT_ROLE_CATEGORY), eq("P_ROLE1"))).thenReturn("Primary Decoded");
        when(codeDecodeHelper.getCodeDecodeShortDesc(eq(PARTICIPANT_ROLE_CATEGORY), eq("A_ROLE1"))).thenReturn("Affected Decoded");

        List<LiabilityParticipantDetailModel> group = Collections.singletonList(mappedParticipant);
        when(liabilityServiceHelper.joinNonNullDistinctStrings(eq(group), any(), eq(EMPTY_STRING))).thenAnswer(invocation -> {
            java.util.function.Function<LiabilityParticipantDetailModel, String> mapper = invocation.getArgument(1);
            if (mapper.apply(group.get(0)).equals(group.get(0).getLiabilityDetails().get(0).getAdjusterLow())) return "50";
            if (mapper.apply(group.get(0)).equals(group.get(0).getLiabilityDetails().get(0).getFinalLiabilityPct())) return "50";
            if (mapper.apply(group.get(0)).equals(group.get(0).getLiabilityDetails().get(0).getFinalLiability())) return "FinalLiabValue";
            return "Mocked String";
        });

        when(service.getFinancialLiability(eq(INITIAL), eq(NEGLigence_RULE), eq("50"), eq("50"), eq(Arrays.asList("P_ROLE1")), eq(Arrays.asList("A_ROLE1"))))
                .thenReturn("CalculatedLiabilityForSingle");
        when(service.isLiabilityModified(eq(INITIAL), eq("50"), eq("50"), eq("CalculatedLiabilityForSingle"))).thenReturn(false);

        LiabilityDetailModel expectedResultModel = new LiabilityDetailModel();
        when(liabilityDetailModelBuilder.build(eq("Status"), eq(INITIAL), any(List.class), eq(NEGLigence_RULE))).thenReturn(expectedResultModel);

        LiabilityDetailModel result = service.getLiabilityDetails(CLAIM_ID, NEGLigence_RULE);

        assertNotNull(result);
        verify(liabilityRepository, times(1)).getAllByClaimId(CLAIM_ID);
        verify(liabilityServiceHelper, times(1)).getStatus(liabilityDataList);
        verify(liabilityServiceHelper, times(1)).getStatusCode("Status");
        verify(liabilityServiceHelper, times(1)).mapToObject(any(LiabilityData.class));
        verify(liabilityServiceHelper, times(1)).getGroupKey(mappedParticipant);
        verify(liabilityServiceHelper, times(1)).joinNonNullDistinctList(any(), any()); // Primary roles
        verify(liabilityServiceHelper, times(1)).joinNonNullDistinctList(any(), any()); // Affected roles
        verify(liabilityServiceHelper, times(3)).joinNonNullDistinctStrings(eq(group), any(), eq(EMPTY_STRING));
        verify(service, times(1)).getFinancialLiability(eq(INITIAL), eq(NEGLigence_RULE), eq("50"), eq("50"), eq(Arrays.asList("P_ROLE1")), eq(Arrays.asList("A_ROLE1")));
        verify(service, times(1)).isLiabilityModified(eq(INITIAL), eq("50"), eq("50"), eq("CalculatedLiabilityForSingle"));
        verify(codeDecodeHelper, times(1)).getCodeDecodeShortDesc(eq(PARTICIPANT_ROLE_CATEGORY), eq("P_ROLE1"));
        verify(codeDecodeHelper, times(1)).getCodeDecodeShortDesc(eq(PARTICIPANT_ROLE_CATEGORY), eq("A_ROLE1"));
        verify(liabilityDetailModelBuilder, times(1)).build(eq("Status"), eq(INITIAL), any(List.class), eq(NEGLigence_RULE));

        List<LiabilityParticipantDetailModel> expectedParticipants = new ArrayList<>();
        AffectedParticipantDetailModel expectedAffected = AffectedParticipantDetailModel.builder()
                .affectedParticipantRole(Arrays.asList("Affected Decoded"))
                .affectedParticipant("Affected1 Name")
                .finalLiability("FinalLiabValue")
                .affectedParticipantId("Affected1")
                .adjusterLow("50")
                .adjusterHigh("Affected1 AdjusterHigh")
                .financialLiability("CalculatedLiabilityForSingle")
                .isLiabilityModified(false)
                .build();
        LiabilityParticipantDetailModel expectedParticipantModel = LiabilityParticipantDetailModel.builder()
                .primaryParticipant("P1 Name")
                .liabilityDetails(Collections.singletonList(expectedAffected))
                .primaryParticipantRole(Arrays.asList("Primary Decoded"))
                .primaryParticipantId("P1")
                .build();
         expectedParticipants.add(expectedParticipantModel);

        verify(liabilityDetailModelBuilder, times(1)).build(eq("Status"), eq(INITIAL), eq(expectedParticipants), eq(NEGLigence_RULE));
    }

     @Test
    void getLiabilityDetails_multipleEntriesSameParticipant() {
        List<LiabilityData> liabilityDataList = Arrays.asList(new LiabilityData(), new LiabilityData());

        LiabilityParticipantDetailModel mappedParticipant1 = createMockParticipant("P1", "Affected1", "Key1", "40", "40", "P_ROLE1", "A_ROLE1");
        LiabilityParticipantDetailModel mappedParticipant2 = createMockParticipant("P1", "Affected2", "Key2", "60", "60", "P_ROLE1", "A_ROLE2");

        when(liabilityRepository.getAllByClaimId(CLAIM_ID)).thenReturn(liabilityDataList);
        when(liabilityServiceHelper.getStatus(liabilityDataList)).thenReturn("Status");
        when(liabilityServiceHelper.getStatusCode("Status")).thenReturn(FINAL);

        when(liabilityServiceHelper.mapToObject(liabilityDataList.get(0))).thenReturn(mappedParticipant1);
        when(liabilityServiceHelper.mapToObject(liabilityDataList.get(1))).thenReturn(mappedParticipant2);
        when(liabilityServiceHelper.getGroupKey(mappedParticipant1)).thenReturn("Key1");
        when(liabilityServiceHelper.getGroupKey(mappedParticipant2)).thenReturn("Key2");

        when(liabilityServiceHelper.joinNonNullDistinctList(any(), any())).thenReturn(
            Arrays.asList("P_ROLE1"), // Primary roles for group 1 ([mappedParticipant1])
            Arrays.asList("A_ROLE1"), // Affected roles for group 1 ([mappedParticipant1])
            Arrays.asList("P_ROLE1"), // Primary roles for group 2 ([mappedParticipant2])
            Arrays.asList("A_ROLE2")  // Affected roles for group 2 ([mappedParticipant2])
        );

         when(liabilityServiceHelper.joinNonNullDistinctStrings(any(), any(), eq(EMPTY_STRING))).thenAnswer(invocation -> {
            List<LiabilityParticipantDetailModel> currentGroup = invocation.getArgument(0);
             java.util.function.Function<LiabilityParticipantDetailModel, String> mapper = invocation.getArgument(1);

            if (currentGroup.get(0).getLiabilityDetails() != null && !currentGroup.get(0).getLiabilityDetails().isEmpty()) {
                 AffectedParticipantDetailModel firstAffected = currentGroup.get(0).getLiabilityDetails().get(0);
                 // Distinguish based on affected ID within the group's first element
                 if ("Affected1".equals(firstAffected.getAffectedParticipantId())) {
                     if (mapper.apply(currentGroup.get(0)).equals(firstAffected.getAdjusterLow())) return "40";
                     if (mapper.apply(currentGroup.get(0)).equals(firstAffected.getFinalLiabilityPct())) return "40";
                     if (mapper.apply(currentGroup.get(0)).equals(firstAffected.getFinalLiability())) return "Affected1 FinalLiabValue";
                 } else if ("Affected2".equals(firstAffected.getAffectedParticipantId())) {
                      if (mapper.apply(currentGroup.get(0)).equals(firstAffected.getAdjusterLow())) return "60";
                     if (mapper.apply(currentGroup.get(0)).equals(firstAffected.getFinalLiabilityPct())) return "60";
                     if (mapper.apply(currentGroup.get(0)).equals(firstAffected.getFinalLiability())) return "Affected2 FinalLiabValue";
                 }
            }
             return "Mocked String Value";
        });

        // Mock getFinancialLiability and isLiabilityModified matching the extracted values
        when(service.getFinancialLiability(eq(FINAL), eq(NEGLigence_RULE), eq("40"), eq("40"), eq(Arrays.asList("P_ROLE1")), eq(Arrays.asList("A_ROLE1")))).thenReturn("FL1");
        when(service.isLiabilityModified(eq(FINAL), eq("40"), eq("40"), eq("FL1"))).thenReturn(false);

        when(service.getFinancialLiability(eq(FINAL), eq(NEGLigence_RULE), eq("60"), eq("60"), eq(Arrays.asList("P_ROLE1")), eq(Arrays.asList("A_ROLE2")))).thenReturn("FL2");
        when(service.isLiabilityModified(eq(FINAL), eq("60"), eq("60"), eq("FL2"))).thenReturn(true);

        when(codeDecodeHelper.getCodeDecodeShortDesc(eq(PARTICIPANT_ROLE_CATEGORY), eq("P_ROLE1"))).thenReturn("Primary Decoded");
        when(codeDecodeHelper.getCodeDecodeShortDesc(eq(PARTICIPANT_ROLE_CATEGORY), eq("A_ROLE1"))).thenReturn("Affected1 Decoded");
        when(codeDecodeHelper.getCodeDecodeShortDesc(eq(PARTICIPANT_ROLE_CATEGORY), eq("A_ROLE2"))).thenReturn("Affected2 Decoded");


        LiabilityDetailModel expectedResultModel = new LiabilityDetailModel();
        when(liabilityDetailModelBuilder.build(eq("Status"), eq(FINAL), any(List.class), eq(NEGLigence_RULE))).thenReturn(expectedResultModel);

        LiabilityDetailModel result = service.getLiabilityDetails(CLAIM_ID, NEGLigence_RULE);

        assertNotNull(result);
        verify(liabilityRepository, times(1)).getAllByClaimId(CLAIM_ID);
        verify(liabilityServiceHelper, times(1)).getStatus(liabilityDataList);
        verify(liabilityServiceHelper, times(1)).getStatusCode("Status");
        verify(liabilityServiceHelper, times(1)).mapToObject(liabilityDataList.get(0));
        verify(liabilityServiceHelper, times(1)).mapToObject(liabilityDataList.get(1));
        verify(liabilityServiceHelper, times(1)).getGroupKey(mappedParticipant1);
        verify(liabilityServiceHelper, times(1)).getGroupKey(mappedParticipant2);

        verify(liabilityServiceHelper, times(4)).joinNonNullDistinctList(any(), any()); // 2 primary + 2 affected calls
        verify(liabilityServiceHelper, times(6)).joinNonNullDistinctStrings(any(), any(), eq(EMPTY_STRING)); // 3 calls per loop iter * 2 iters = 6

        verify(service, times(1)).getFinancialLiability(eq(FINAL), eq(NEGLigence_RULE), eq("40"), eq("40"), eq(Arrays.asList("P_ROLE1")), eq(Arrays.asList("A_ROLE1")));
        verify(service, times(1)).getFinancialLiability(eq(FINAL), eq(NEGLigence_RULE), eq("60"), eq("60"), eq(Arrays.asList("P_ROLE1")), eq(Arrays.asList("A_ROLE2")));
        verify(service, times(1)).isLiabilityModified(eq(FINAL), eq("40"), eq("40"), eq("FL1"));
        verify(service, times(1)).isLiabilityModified(eq(FINAL), eq("60"), eq("60"), eq("FL2"));

        verify(codeDecodeHelper, times(2)).getCodeDecodeShortDesc(eq(PARTICIPANT_ROLE_CATEGORY), eq("P_ROLE1"));
        verify(codeDecodeHelper, times(1)).getCodeDecodeShortDesc(eq(PARTICIPANT_ROLE_CATEGORY), eq("A_ROLE1"));
        verify(codeDecodeHelper, times(1)).getCodeDecodeShortDesc(eq(PARTICIPANT_ROLE_CATEGORY), eq("A_ROLE2"));

        List<LiabilityParticipantDetailModel> expectedParticipants = new ArrayList<>();
        AffectedParticipantDetailModel expectedAffected1 = AffectedParticipantDetailModel.builder()
                .affectedParticipantRole(Arrays.asList("Affected1 Decoded"))
                .affectedParticipant("Affected1 Name")
                .finalLiability("Affected1 FinalLiabValue")
                .affectedParticipantId("Affected1")
                .adjusterLow("40")
                .adjusterHigh("Affected1 AdjusterHigh")
                .financialLiability("FL1")
                .isLiabilityModified(false)
                .build();
         AffectedParticipantDetailModel expectedAffected2 = AffectedParticipantDetailModel.builder()
                .affectedParticipantRole(Arrays.asList("Affected2 Decoded"))
                .affectedParticipant("Affected2 Name")
                .finalLiability("Affected2 FinalLiabValue")
                .affectedParticipantId("Affected2")
                .adjusterLow("60")
                .adjusterHigh("Affected2 AdjusterHigh")
                .financialLiability("FL2")
                .isLiabilityModified(true)
                .build();

        LiabilityParticipantDetailModel expectedParticipantModel = LiabilityParticipantDetailModel.builder()
                .primaryParticipant("P1 Name")
                .liabilityDetails(Arrays.asList(expectedAffected1, expectedAffected2))
                .primaryParticipantRole(Arrays.asList("Primary Decoded"))
                .primaryParticipantId("P1")
                .build();
         expectedParticipants.add(expectedParticipantModel);

        verify(liabilityDetailModelBuilder, times(1)).build(eq("Status"), eq(FINAL), eq(expectedParticipants), eq(NEGLigence_RULE));
    }

     @Test
    void getLiabilityDetails_multipleEntriesDifferentParticipants() {
        List<LiabilityData> liabilityDataList = Arrays.asList(new LiabilityData(), new LiabilityData());

        LiabilityParticipantDetailModel mappedParticipant1 = createMockParticipant("P1", "Affected1", "Key1", "40", "40", "P_ROLE1", "A_ROLE1");
        LiabilityParticipantDetailModel mappedParticipant2 = createMockParticipant("P2", "Affected2", "Key2", "60", "60", "P_ROLE2", "A_ROLE2");

        when(liabilityRepository.getAllByClaimId(CLAIM_ID)).thenReturn(liabilityDataList);
        when(liabilityServiceHelper.getStatus(liabilityDataList)).thenReturn("Status");
        when(liabilityServiceHelper.getStatusCode("Status")).thenReturn(FINAL);

        when(liabilityServiceHelper.mapToObject(liabilityDataList.get(0))).thenReturn(mappedParticipant1);
        when(liabilityServiceHelper.mapToObject(liabilityDataList.get(1))).thenReturn(mappedParticipant2);
        when(liabilityServiceHelper.getGroupKey(mappedParticipant1)).thenReturn("Key1");
        when(liabilityServiceHelper.getGroupKey(mappedParticipant2)).thenReturn("Key2");

         // Corrected mock for joinNonNullDistinctList - using sequential returns for different participant groups
         when(liabilityServiceHelper.joinNonNullDistinctList(any(), any())).thenReturn(
            Arrays.asList("P_ROLE1"), // 1st call: primary roles for [mappedParticipant1]
            Arrays.asList("A_ROLE1"), // 2nd call: affected roles for [mappedParticipant1]
            Arrays.asList("P_ROLE2"), // 3rd call: primary roles for [mappedParticipant2]
            Arrays.asList("A_ROLE2")  // 4th call: affected roles for [mappedParticipant2]
         );

        // Mocking joinNonNullDistinctStrings - distinguishing groups by primary ID in the group
         when(liabilityServiceHelper.joinNonNullDistinctStrings(any(), any(), eq(EMPTY_STRING))).thenAnswer(invocation -> {
            List<LiabilityParticipantDetailModel> currentGroup = invocation.getArgument(0);
             java.util.function.Function<LiabilityParticipantDetailModel, String> mapper = invocation.getArgument(1);

             LiabilityParticipantDetailModel participantInGroup = currentGroup.get(0);
             String primaryId = participantInGroup.getPrimaryParticipantId();

             if (participantInGroup.getLiabilityDetails() != null && !participantInGroup.getLiabilityDetails().isEmpty()) {
                 AffectedParticipantDetailModel firstAffected = participantInGroup.getLiabilityDetails().get(0);

                if ("P1".equals(primaryId)) {
                     if (mapper.apply(participantInGroup).equals(firstAffected.getAdjusterLow())) return "40";
                     if (mapper.apply(participantInGroup).equals(firstAffected.getFinalLiabilityPct())) return "40";
                     if (mapper.apply(participantInGroup).equals(firstAffected.getFinalLiability())) return "Affected1 FinalLiabValue";
                } else if ("P2".equals(primaryId)) {
                     if (mapper.apply(participantInGroup).equals(firstAffected.getAdjusterLow())) return "60";
                     if (mapper.apply(participantInGroup).equals(firstAffected.getFinalLiabilityPct())) return "60";
                     if (mapper.apply(participantInGroup).equals(firstAffected.getFinalLiability())) return "Affected2 FinalLiabValue";
                }
             }
             return "Mocked String Value";
        });


        // Mock getFinancialLiability and isLiabilityModified matching the extracted values and roles
        when(service.getFinancialLiability(eq(FINAL), eq(NEGLigence_RULE), eq("40"), eq("40"), eq(Arrays.asList("P_ROLE1")), eq(Arrays.asList("A_ROLE1")))).thenReturn("FL1");
        when(service.isLiabilityModified(eq(FINAL), eq("40"), eq("40"), eq("FL1"))).thenReturn(false);

        when(service.getFinancialLiability(eq(FINAL), eq(NEGLigence_RULE), eq("60"), eq("60"), eq(Arrays.asList("P_ROLE2")), eq(Arrays.asList("A_ROLE2")))).thenReturn("FL2");
        when(service.isLiabilityModified(eq(FINAL), eq("60"), eq("60"), eq("FL2"))).thenReturn(true);


        when(codeDecodeHelper.getCodeDecodeShortDesc(eq(PARTICIPANT_ROLE_CATEGORY), eq("P_ROLE1"))).thenReturn("Primary1 Decoded");
        when(codeDecodeHelper.getCodeDecodeShortDesc(eq(PARTICIPANT_ROLE_CATEGORY), eq("A_ROLE1"))).thenReturn("Affected1 Decoded");
        when(codeDecodeHelper.getCodeDecodeShortDesc(eq(PARTICIPANT_ROLE_CATEGORY), eq("P_ROLE2"))).thenReturn("Primary2 Decoded");
        when(codeDecodeHelper.getCodeDecodeShortDesc(eq(PARTICIPANT_ROLE_CATEGORY), eq("A_ROLE2"))).thenReturn("Affected2 Decoded");


        LiabilityDetailModel expectedResultModel = new LiabilityDetailModel();
        when(liabilityDetailModelBuilder.build(eq("Status"), eq(FINAL), any(List.class), eq(NEGLigence_RULE))).thenReturn(expectedResultModel);


        LiabilityDetailModel result = service.getLiabilityDetails(CLAIM_ID, NEGLigence_RULE);

        assertNotNull(result);
        verify(liabilityRepository, times(1)).getAllByClaimId(CLAIM_ID);
        verify(liabilityServiceHelper, times(1)).getStatus(liabilityDataList);
        verify(liabilityServiceHelper, times(1)).getStatusCode("Status");
        verify(liabilityServiceHelper, times(1)).mapToObject(liabilityDataList.get(0));
        verify(liabilityServiceHelper, times(1)).mapToObject(liabilityDataList.get(1));
        verify(liabilityServiceHelper, times(1)).getGroupKey(mappedParticipant1);
        verify(liabilityServiceHelper, times(1)).getGroupKey(mappedParticipant2);

        verify(liabilityServiceHelper, times(4)).joinNonNullDistinctList(any(), any());
        verify(liabilityServiceHelper, times(6)).joinNonNullDistinctStrings(any(), any(), eq(EMPTY_STRING));

        verify(service, times(1)).getFinancialLiability(eq(FINAL), eq(NEGLigence_RULE), eq("40"), eq("40"), eq(Arrays.asList("P_ROLE1")), eq(Arrays.asList("A_ROLE1")));
        verify(service, times(1)).getFinancialLiability(eq(FINAL), eq(NEGLigence_RULE), eq("60"), eq("60"), eq(Arrays.asList("P_ROLE2")), eq(Arrays.asList("A_ROLE2")));
        verify(service, times(1)).isLiabilityModified(eq(FINAL), eq("40"), eq("40"), eq("FL1"));
        verify(service, times(1)).isLiabilityModified(eq(FINAL), eq("60"), eq("60"), eq("FL2"));

        verify(codeDecodeHelper, times(1)).getCodeDecodeShortDesc(eq(PARTICIPANT_ROLE_CATEGORY), eq("P_ROLE1"));
        verify(codeDecodeHelper, times(1)).getCodeDecodeShortDesc(eq(PARTICIPANT_ROLE_CATEGORY), eq("A_ROLE1"));
        verify(codeDecodeHelper, times(1)).getCodeDecodeShortDesc(eq(PARTICIPANT_ROLE_CATEGORY), eq("P_ROLE2"));
        verify(codeDecodeHelper, times(1)).getCodeDecodeShortDesc(eq(PARTICIPANT_ROLE_CATEGORY), eq("A_ROLE2"));


        List<LiabilityParticipantDetailModel> expectedParticipants = new ArrayList<>();
        AffectedParticipantDetailModel expectedAffected1 = AffectedParticipantDetailModel.builder()
                .affectedParticipantRole(Arrays.asList("Affected1 Decoded"))
                .affectedParticipant("Affected1 Name")
                .finalLiability("Affected1 FinalLiabValue")
                .affectedParticipantId("Affected1")
                .adjusterLow("40")
                .adjusterHigh("Affected1 AdjusterHigh")
                .financialLiability("FL1")
                .isLiabilityModified(false)
                .build();
        LiabilityParticipantDetailModel expectedParticipantModel1 = LiabilityParticipantDetailModel.builder()
                .primaryParticipant("P1 Name")
                .liabilityDetails(Collections.singletonList(expectedAffected1))
                .primaryParticipantRole(Arrays.asList("Primary1 Decoded"))
                .primaryParticipantId("P1")
                .build();
         expectedParticipants.add(expectedParticipantModel1);

         AffectedParticipantDetailModel expectedAffected2 = AffectedParticipantDetailModel.builder()
                .affectedParticipantRole(Arrays.asList("Affected2 Decoded"))
                .affectedParticipant("Affected2 Name")
                .finalLiability("Affected2 FinalLiabValue")
                .affectedParticipantId("Affected2")
                .adjusterLow("60")
                .adjusterHigh("Affected2 AdjusterHigh")
                .financialLiability("FL2")
                .isLiabilityModified(true)
                .build();
        LiabilityParticipantDetailModel expectedParticipantModel2 = LiabilityParticipantDetailModel.builder()
                .primaryParticipant("P2 Name")
                .liabilityDetails(Collections.singletonList(expectedAffected2))
                .primaryParticipantRole(Arrays.asList("Primary2 Decoded"))
                .primaryParticipantId("P2")
                .build();
        expectedParticipants.add(expectedParticipantModel2);


        verify(liabilityDetailModelBuilder, times(1)).build(eq("Status"), eq(FINAL), eq(expectedParticipants), eq(NEGLigence_RULE));
    }

    @Test
    void shouldGetInitialLiabilityR01() {
        when(helper.isBlankOrNull(any())).thenReturn(false);
        when(helper.isBlankOrNull("")).thenReturn(true);
        when(helper.isBlankOrNull(null)).thenReturn(true);

         List<String> list_PIN_OWNR = Arrays.asList(PIN, OWNR);
         List<String> list_CMT_OWNR = Arrays.asList(CMT, OWNR);
         List<String> list_PIN = Arrays.asList(PIN);
         List<String> list_CMT = Arrays.asList(CMT);
         List<String> list_OWNR = Arrays.asList(OWNR);
         List<String> list_Other = Arrays.asList("OTHER");
         List<String> emptyList = Collections.emptyList();

        String initialLiability1 = service.getLiability(Contributory_Negligence, "99", list_PIN_OWNR, list_CMT_OWNR);
        String initialLiability2 = service.getLiability(Contributory_Negligence, "100", list_PIN_OWNR, list_CMT_OWNR);
        String initialLiability3 = service.getLiability(Contributory_Negligence, "35", list_PIN, list_CMT);
        String initialLiability4 = service.getLiability(Contributory_Negligence, "35", list_OWNR, list_OWNR);
        String initialLiability5 = service.getLiability(Contributory_Negligence, "35", emptyList, Arrays.asList("", ""));

        String initialLiability6 = service.getLiability(Pure_Comparative, "35", list_Other, emptyList);
        String initialLiability7 = service.getLiability(Contributory_Negligence, "100", list_Other, emptyList);
        String initialLiability8 = service.getLiability(Contributory_Negligence, "50", list_Other, emptyList);

        String initialLiability9 = service.getLiability(Pure_Comparative, "40", list_PIN_OWNR, Arrays.asList(OWNR));
        String initialLiability10 = service.getLiability(Pure_Comparative, "30", list_PIN_OWNR, Arrays.asList("ADDIN"));

        String initialLiability11 = service.getLiability(Pure_Comparative, "abc", list_Other, emptyList);

        assertEquals("", initialLiability1);
        assertEquals("", initialLiability2);
        assertEquals("", initialLiability3);
        assertEquals("", initialLiability4);
        assertEquals("", initialLiability5);
        assertEquals("35", initialLiability6);
        assertEquals("100", initialLiability7);
        assertEquals("0", initialLiability8);
        assertEquals("", initialLiability9);
        assertEquals("", initialLiability10);
        assertEquals("0", initialLiability11);
    }

    @Test
    void shouldGetClaimantLiabilityR01() {
        when(helper.isBlankOrNull(any())).thenReturn(false);
        when(helper.isBlankOrNull("")).thenReturn(true);
        when(helper.isBlankOrNull(null)).thenReturn(true);

         List<String> list_PIN_OWNR = Arrays.asList(PIN, OWNR);
         List<String> list_CMT_OWNR = Arrays.asList(CMT, OWNR);
         List<String> list_CMT = Arrays.asList(CMT);
         List<String> list_OWNR = Arrays.asList(OWNR);
         List<String> list_Other = Arrays.asList("OTHER");
         List<String> emptyList = Collections.emptyList();

        String claimantLiability1 = service.getLiability(Contributory_Negligence, "99", list_CMT_OWNR, list_PIN_OWNR);
        String claimantLiability2 = service.getLiability(Contributory_Negligence, "100", list_CMT_OWNR, list_PIN_OWNR);
        String claimantLiability3 = service.getLiability(Contributory_Negligence, "35", list_CMT, list_PIN_OWNR);
        String claimantLiability4 = service.getLiability(Contributory_Negligence, "35", list_OWNR, list_OWNR);
        String claimantLiability5 = service.getLiability(Contributory_Negligence, "35", emptyList, Arrays.asList("", ""));

        String claimantLiability6 = service.getLiability(Pure_Comparative, "35", list_Other, emptyList);
        String claimantLiability7 = service.getLiability(Contributory_Negligence, "100", list_Other, emptyList);


        assertEquals("", claimantLiability1);
        assertEquals("", claimantLiability2);
        assertEquals("", claimantLiability3);
        assertEquals("", claimantLiability4);
        assertEquals("", claimantLiability5);
        assertEquals("35", claimantLiability6);
        assertEquals("100", claimantLiability7);
    }

    @Test
    void shouldHandleEmptyNegligenceRuleAndLiability() {
        when(helper.isBlankOrNull("")).thenReturn(true);
        when(helper.isBlankOrNull(null)).thenReturn(true);
         List<String> otherRoles = Arrays.asList("OTHER");

        String initialLiability = service.getLiability(null, "", otherRoles, otherRoles);
        String initialLiability1 = service.getLiability("", "", otherRoles, otherRoles);
        String initialLiability2 = service.getLiability(Contributory_Negligence, "", otherRoles, otherRoles);
        String initialLiability3 = service.getLiability(Contributory_Negligence, null, otherRoles, otherRoles);

        assertEquals("", initialLiability);
        assertEquals("", initialLiability1);
        assertEquals("", initialLiability2);
        assertEquals("", initialLiability3);
    }

    @Test
    void shouldGetLiabilityR02AndR05() {
        when(helper.isBlankOrNull(any())).thenReturn(false);
        List<String> otherRoles = Arrays.asList("OTHER");

        String initialLiability1 = service.getLiability(Pure_Comparative, "49", otherRoles, otherRoles);
        String initialLiability2 = service.getLiability(Other, "50", otherRoles, otherRoles);

        assertEquals("49", initialLiability1);
        assertEquals("50", initialLiability2);
    }

    @Test
    void shouldGetLiabilityR03() {
        when(helper.isBlankOrNull(any())).thenReturn(false);
        List<String> otherRoles = Arrays.asList("OTHER");

        String initialLiability1 = service.getLiability(Mod_Comparative_50, "49", otherRoles, otherRoles);
        String initialLiability2 = service.getLiability(Mod_Comparative_50, "50", otherRoles, otherRoles);
        String initialLiability3 = service.getLiability(Mod_Comparative_50, "51", otherRoles, otherRoles);
        String initialLiability4 = service.getLiability(Mod_Comparative_50, "100", otherRoles, otherRoles);

        assertEquals("0", initialLiability1);
        assertEquals("0", initialLiability2);
        assertEquals("51", initialLiability3);
        assertEquals("100", initialLiability4);
    }

    @Test
    void shouldGetLiabilityR04() {
        when(helper.isBlankOrNull(any())).thenReturn(false);
        List<String> otherRoles = Arrays.asList("OTHER");

        String initialLiability1 = service.getLiability(Mod_Comparative_49, "49", otherRoles, otherRoles);
        String initialLiability2 = service.getLiability(Mod_Comparative_49, "50", otherRoles, otherRoles);
        String initialLiability3 = service.getLiability(Mod_Comparative_49, "100", otherRoles, otherRoles);

        assertEquals("0", initialLiability1);
        assertEquals("50", initialLiability2);
        assertEquals("100", initialLiability3);
    }

    @Test
    void shouldGetLiabilityR06() {
        when(helper.isBlankOrNull(any())).thenReturn(false);
        List<String> otherRoles = Arrays.asList("OTHER");

        String initialLiability1 = service.getLiability(Mod_Comparative_51, "51", otherRoles, otherRoles);
        String initialLiability2 = service.getLiability(Mod_Comparative_51, "52", otherRoles, otherRoles);
        String initialLiability3 = service.getLiability(Mod_Comparative_51, "100", otherRoles, otherRoles);
        String initialLiability4 = service.getLiability(Mod_Comparative_51, "49", otherRoles, otherRoles);

        assertEquals("51", initialLiability1);
        assertEquals("52", initialLiability2);
        assertEquals("100", initialLiability3);
        assertEquals("0", initialLiability4);
    }

    @Test
    void shouldGetLiabilityR07() {
        when(helper.isBlankOrNull(any())).thenReturn(false);
        List<String> otherRoles = Arrays.asList("OTHER");

        String initialLiability1 = service.getLiability(Slight_comparative, "79", otherRoles, otherRoles);
        String initialLiability2 = service.getLiability(Slight_comparative, "80", otherRoles, otherRoles);
        String initialLiability3 = service.getLiability(Slight_comparative, "100", otherRoles, otherRoles);

        assertEquals("0", initialLiability1);
        assertEquals("80", initialLiability2);
        assertEquals("100", initialLiability3);
    }

    @Test
    void shouldGetFinancialLiability() {
        when(service.getLiability(any(), any(), any(), any())).thenReturn("CalculatedFromGetLiability");

        String result = service.getFinancialLiability("","","","", Arrays.asList("",""), Arrays.asList("",""));
        String result1 = service.getFinancialLiability(null,"","","", Arrays.asList("",""), Arrays.asList("",""));
        String resultWithNegligenceRuleNull = service.getFinancialLiability("final",null,"82","80", primaryParticipantList, affectedParticipantList);

        String initalStatus = service.getFinancialLiability("initial",Slight_comparative,"82","80", primaryParticipantList, affectedParticipantList);
        String finalStatus = service.getFinancialLiability("final",Slight_comparative,"82","80", primaryParticipantList, affectedParticipantList);

        assertEquals("", result);
        assertEquals("", result1);
        assertEquals("", resultWithNegligenceRuleNull);
        assertEquals("CalculatedFromGetLiability", initalStatus);
        assertEquals("CalculatedFromGetLiability", finalStatus);

        verify(service, times(1)).getLiability(eq(Slight_comparative), eq("82"), eq(primaryParticipantList), eq(affectedParticipantList));
        verify(service, times(1)).getLiability(eq(Slight_comparative), eq("80"), eq(primaryParticipantList), eq(affectedParticipantList));
    }

    @Test
    void isLiabilityModifiedReturnsTrueFalseWhenStatusIsInitialAndComparingAdjusterFinancialLiabilityAndAdjusterLow() {
        boolean resultInitial = service.isLiabilityModified(INITIAL, "50", null, "60");
        boolean resultInitial1 = service.isLiabilityModified(INITIAL, "50", null, "50");
        boolean resultInitial2 = service.isLiabilityModified(INITIAL, "50", null, null);
        boolean resultInitial3 = service.isLiabilityModified(INITIAL, null, null, "50");
        boolean resultInitial4 = service.isLiabilityModified(INITIAL, null, null, null);

        assertTrue(resultInitial);
        assertFalse(resultInitial1);
        assertTrue(resultInitial2);
        assertTrue(resultInitial3);
        assertFalse(resultInitial4);
    }

    @Test
    void isLiabilityModifiedReturnsTrueFalseWhenStatusIsFinalAndComparingFinalFinancialLiabilityAndFinalLiabilityPct() {
        boolean resultFinal = service.isLiabilityModified(FINAL, null, "70", "80");
        boolean resultFinal1 = service.isLiabilityModified(FINAL, null, "70", "70");
        boolean resultFinal2 = service.isLiabilityModified(FINAL, null, "70", null);
        boolean resultFinal3 = service.isLiabilityModified(FINAL, null, null, "70");
        boolean resultFinal4 = service.isLiabilityModified(FINAL, null, null, null);

        assertTrue(resultFinal);
        assertFalse(resultFinal1);
        assertTrue(resultFinal2);
        assertTrue(resultFinal3);
        assertFalse(resultFinal4);
    }

     @Test
     void isLiabilityModifiedReturnsFalseForOtherStatuses() {
         boolean resultOtherStatus = service.isLiabilityModified("OTHER_STATUS", "50", "70", "60");
         assertFalse(resultOtherStatus);
     }

    private LiabilityParticipantDetailModel createMockParticipant(
            String primaryId, String affectedId, String groupKey,
            String adjusterLow, String finalLiabilityPct,
            String primaryRole, String affectedRole) {

        AffectedParticipantDetailModel affectedDetail = AffectedParticipantDetailModel.builder()
                .affectedParticipant("Affected" + affectedId + " Name")
                .affectedParticipantId(affectedId)
                .affectedParticipantRole(Arrays.asList(affectedRole))
                .finalLiability("Affected" + affectedId + " FinalLiabValue")
                .adjusterLow(adjusterLow)
                .adjusterHigh("Affected" + affectedId + " AdjusterHigh")
                .financialLiability("")
                .isLiabilityModified(false)
                .build();

        List<AffectedParticipantDetailModel> affectedDetails = new ArrayList<>();
        affectedDetails.add(affectedDetail);

        LiabilityParticipantDetailModel participantModel = LiabilityParticipantDetailModel.builder()
                .primaryParticipantId(primaryId)
                .primaryParticipant("P" + primaryId + " Name")
                .primaryParticipantRole(Arrays.asList(primaryRole))
                .liabilityDetails(affectedDetails)
                .build();

        return participantModel;
    }
}
