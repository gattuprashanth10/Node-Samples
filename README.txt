@BeforeEach
public void setUp(){
    primaryParticipantList = Arrays.asList(PIN, OWNR);
    affectedParticipantList = Arrays.asList(CMT, OWNR);
    primaryParticipantCMTList = Arrays.asList(CMT, OWNR);
    affectedParticipantCMTList = Arrays.asList(PIN, OWNR);

    // Mock data for roles to trigger the joinNonNullDistinctList and joinNonNullDistinctStrings methods
    List<String> primaryParticipantRoles = Arrays.asList("PrimaryRole1", "PrimaryRole2");
    List<String> affectedParticipantRoles = Arrays.asList("AffectedRole1", "AffectedRole2");
    String adjusterLow = "AdjusterLow";
    String finalLiabilityPct = "100";

    // Create mock LiabilityParticipantDetailModel with roles and liability details populated
    LiabilityParticipantDetailModel liabilityParticipantDetail = getLiabilityParticipantDetailModelWithRoles(
            "123", "test", "affected", primaryParticipantRoles, affectedParticipantRoles, adjusterLow, finalLiabilityPct);

    // Mock behavior to return the list with the roles and details
    when(liabilityServiceHelper.joinNonNullDistinctList(any(), any())).thenReturn(primaryParticipantRoles, affectedParticipantRoles);
    when(liabilityServiceHelper.joinNonNullDistinctStrings(any(), any(), any())).thenReturn(adjusterLow, finalLiabilityPct);
    when(liabilityServiceHelper.mapToObject(Mockito.any())).thenReturn(liabilityParticipantDetail);
    when(liabilityRepository.getAllByClaimId(claimId)).thenReturn(Collections.singletonList(new LiabilityData()));
    when(liabilityServiceHelper.getGroupKey(Mockito.any())).thenReturn("Key");
    when(liabilityDetailModelBuilder.build(Mockito.any(), Mockito.any(), Mockito.any(), any())).thenReturn(new LiabilityDetailModel());
}

// Helper method to create LiabilityParticipantDetailModel with roles
private LiabilityParticipantDetailModel getLiabilityParticipantDetailModelWithRoles(
        String primaryId, String primaryName, String affectedId, 
        List<String> primaryParticipantRoles, List<String> affectedParticipantRoles,
        String adjusterLow, String finalLiabilityPct) {

    AffectedParticipantDetailModel affectedParticipantDetailModel = getAffectedParticipantDetailModel(affectedId, affectedParticipantRoles, adjusterLow, finalLiabilityPct);

    return LiabilityParticipantDetailModel.builder()
            .primaryParticipantId(primaryId)
            .primaryParticipant(primaryName)
            .primaryParticipantRole(primaryParticipantRoles)
            .liabilityDetails(Collections.singletonList(affectedParticipantDetailModel))
            .build();
}

// Helper method to create AffectedParticipantDetailModel with roles, adjusterLow, and finalLiabilityPct
private static AffectedParticipantDetailModel getAffectedParticipantDetailModel(
        String affectedId, List<String> affectedParticipantRoles, String adjusterLow, String finalLiabilityPct) {
    return AffectedParticipantDetailModel.builder()
            .affectedParticipant("affected")
            .affectedParticipantId(affectedId)
            .affectedParticipantRole(affectedParticipantRoles)
            .adjusterLow(adjusterLow)
            .finalLiabilityPct(finalLiabilityPct)
            .financialLiability("")
            .isLiabilityModified(false)
            .build();
}

@Test
public void testGetLiabilityDetailsWithRoles() {
    List<LiabilityData> liabilityDataList = Collections.singletonList(new LiabilityData());
    LiabilityParticipantDetailModel liabilityParticipantDetail = getLiabilityParticipantDetailModelWithRoles(
            "123", "test", "affected", 
            Arrays.asList("PrimaryRole1", "PrimaryRole2"), 
            Arrays.asList("AffectedRole1", "AffectedRole2"), 
            "AdjusterLow", "100");

    when(liabilityRepository.getAllByClaimId(claimId)).thenReturn(liabilityDataList);
    when(liabilityServiceHelper.mapToObject(Mockito.any())).thenReturn(liabilityParticipantDetail);
    when(liabilityServiceHelper.getGroupKey(Mockito.any())).thenReturn("Key");
    when(liabilityServiceHelper.joinNonNullDistinctStrings(Mockito.any(), Mockito.any(), Mockito.any())).thenReturn("AdjusterLow", "100");
    when(liabilityDetailModelBuilder.build(Mockito.any(), Mockito.any(), Mockito.any(), any())).thenReturn(new LiabilityDetailModel());

    // Run the method
    LiabilityDetailModel result = service.getLiabilityDetails(claimId, any());

    // Assertions
    assertNotNull(result);
    verify(liabilityServiceHelper, times(1)).joinNonNullDistinctList(any(), any());
    verify(liabilityServiceHelper, times(1)).joinNonNullDistinctStrings(any(), any(), any());
    assertEquals("AdjusterLow", result.getFinancialLiability()); // or another relevant assertion for your method
}
