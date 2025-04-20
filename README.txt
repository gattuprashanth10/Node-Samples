@Test
public void testGetLiabilityDetails1() {
    List<LiabilityData> liabilityDataList = Collections.singletonList(new LiabilityData());
    when(liabilityRepository.getAllByClaimId(claimId)).thenReturn(liabilityDataList);
    when(liabilityServiceHelper.mapToObject(Mockito.any())).thenReturn(getLiablilityParticipantDetails("123", "test", "affteced"));
    when(liabilityServiceHelper.getGroupKey(Mockito.any())).thenReturn("Key");
    try (MockedStatic<LiabilityServiceHelper> mocked = Mockito.mockStatic(LiabilityServiceHelper.class)) {
        mocked.when(() -> LiabilityServiceHelper.joinNonNullDistinctStrings(Mockito.any(), Mockito.any(), Mockito.any()))
              .thenReturn("Roles");
        mocked.when(() -> LiabilityServiceHelper.joinNonNullDistinctList(Mockito.any(), Mockito.any(), Mockito.any()))
              .thenReturn(Arrays.asList("Role1", "Role2"));
    }
    when(liabilityDetailModelBuilder.build(Mockito.any(),Mockito.any(), Mockito.any(), any())).thenReturn(new LiabilityDetailModel());
    LiabilityDetailModel result = service.getLiabilityDetails(claimId, any());
    assertNotNull(result);
}

@Test
public void testGetLiabilityDetails11() {
    List<LiabilityData> liabilityDataList = Arrays.asList(new LiabilityData(), new LiabilityData());
    LiabilityParticipantDetailModel liablilityParticipantDetails = getLiablilityParticipantDetails("123", "test", "affected");
    liablilityParticipantDetails.getLiabilityDetails().add(getAffectedParticipantDetailModel("affected1"));
    when(liabilityRepository.getAllByClaimId(claimId)).thenReturn(liabilityDataList);
    when(liabilityServiceHelper.mapToObject(Mockito.any()))
            .thenReturn(getLiablilityParticipantDetails("123", "test", "affected"))
            .thenReturn(getLiablilityParticipantDetails("123", "test-1", "affected1"));
    when(liabilityServiceHelper.getGroupKey(Mockito.any())).thenReturn("Key-1").thenReturn("Key-2");
    try (MockedStatic<LiabilityServiceHelper> mocked = Mockito.mockStatic(LiabilityServiceHelper.class)) {
        mocked.when(() -> LiabilityServiceHelper.joinNonNullDistinctStrings(Mockito.any(), Mockito.any(), Mockito.any()))
              .thenReturn("Roles");
        mocked.when(() -> LiabilityServiceHelper.joinNonNullDistinctList(Mockito.any(), Mockito.any(), Mockito.any()))
              .thenReturn(Arrays.asList("Role1", "Role2"));
    }
    when(liabilityDetailModelBuilder.build(Mockito.any(), Mockito.any(), Mockito.any(), any())).thenReturn(new LiabilityDetailModel());
    LiabilityDetailModel result = service.getLiabilityDetails(claimId, any());
    assertNotNull(result);
    verify(liabilityDetailModelBuilder, times(1))
            .build(null, null,
                    Collections.singletonList(liablilityParticipantDetails), null);
}
