@Test
void testJoinNonNullDistinctListCoverage() {
    List<LiabilityData> liabilityDataList = Collections.singletonList(new LiabilityData());
    when(liabilityRepository.getAllByClaimId(claimId)).thenReturn(liabilityDataList);
    when(liabilityServiceHelper.mapToObject(Mockito.any())).thenReturn(getLiablilityParticipantDetails("123", "test", "affected"));
    when(liabilityServiceHelper.getGroupKey(Mockito.any())).thenReturn("Key");
    when(liabilityServiceHelper.joinNonNullDistinctList(Mockito.any(), Mockito.any(), Mockito.any())).thenReturn(Arrays.asList("Role1", "Role2"));
    when(liabilityServiceHelper.joinNonNullDistinctStrings(Mockito.any(), Mockito.any(), Mockito.any())).thenReturn("CombinedRoles");
    when(liabilityDetailModelBuilder.build(Mockito.any(), Mockito.any(), Mockito.any(), any())).thenReturn(new LiabilityDetailModel());

    LiabilityDetailModel result = service.getLiabilityDetails(claimId, any());
    assertNotNull(result);
}

@Test
void testIsLiabilityModifiedInBuilder() {
    List<LiabilityData> liabilityDataList = Collections.singletonList(new LiabilityData());
    AffectedParticipantDetailModel affected = AffectedParticipantDetailModel.builder()
        .affectedParticipant("affected")
        .affectedParticipantId("affected-id")
        .affectedParticipantRole(Arrays.asList("role"))
        .finalLiability("50")
        .financialLiability("60")
        .build();

    LiabilityParticipantDetailModel participant = LiabilityParticipantDetailModel.builder()
        .primaryParticipantId("123")
        .primaryParticipant("test")
        .primaryParticipantRole(Arrays.asList("role"))
        .liabilityDetails(Arrays.asList(affected))
        .build();

    when(liabilityRepository.getAllByClaimId(claimId)).thenReturn(Collections.singletonList(new LiabilityData()));
    when(liabilityServiceHelper.mapToObject(Mockito.any())).thenReturn(participant);
    when(liabilityServiceHelper.getGroupKey(Mockito.any())).thenReturn("GroupKey");
    when(liabilityServiceHelper.joinNonNullDistinctStrings(Mockito.any(), Mockito.any(), Mockito.any())).thenReturn("role");
    when(liabilityServiceHelper.joinNonNullDistinctList(Mockito.any(), Mockito.any(), Mockito.any())).thenReturn(Arrays.asList("role"));
    when(liabilityDetailModelBuilder.build(Mockito.any(), Mockito.any(), Mockito.any(), any())).thenReturn(new LiabilityDetailModel());

    LiabilityDetailModel result = service.getLiabilityDetails(claimId, INITIAL);
    assertNotNull(result);
}
