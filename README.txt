@Test
void testGetLiabilityDetailsWhenRepositoryReturnsEmptyList() {
    when(liabilityRepository.getAllByClaimId(claimId)).thenReturn(Collections.emptyList());

    LiabilityDetailModel result = service.getLiabilityDetails(claimId, any());
    assertNotNull(result);
    assertTrue(result.getLiabilityParticipantDetails().isEmpty()); // Assuming your model initializes with empty list
}


@Test
void testGetLiabilityDetailsWithDuplicateGroupKeySkipsBuild() {
    List<LiabilityData> liabilityDataList = Arrays.asList(new LiabilityData(), new LiabilityData());

    LiabilityParticipantDetailModel participantDetail1 = getLiablilityParticipantDetails("123", "test", "affected");
    LiabilityParticipantDetailModel participantDetail2 = getLiablilityParticipantDetails("124", "test-2", "affected-2");

    when(liabilityRepository.getAllByClaimId(claimId)).thenReturn(liabilityDataList);
    when(liabilityServiceHelper.mapToObject(Mockito.any()))
        .thenReturn(participantDetail1)
        .thenReturn(participantDetail2);
    when(liabilityServiceHelper.getGroupKey(Mockito.any())).thenReturn("DUPLICATE_KEY");
    when(liabilityServiceHelper.joinNonNullDistinctStrings(Mockito.any(), Mockito.any(), Mockito.any())).thenReturn("Roles");

    when(liabilityDetailModelBuilder.build(Mockito.any(), Mockito.any(), Mockito.any(), any()))
        .thenReturn(new LiabilityDetailModel());

    LiabilityDetailModel result = service.getLiabilityDetails(claimId, any());
    assertNotNull(result);
    // Since second key is duplicate, build should only be called once
    verify(liabilityDetailModelBuilder, times(1)).build(any(), any(), any(), any());
}


@Test
void testGetLiabilityDetailsWithNullParticipantDetailsSkipsBuild() {
    List<LiabilityData> liabilityDataList = Collections.singletonList(new LiabilityData());

    when(liabilityRepository.getAllByClaimId(claimId)).thenReturn(liabilityDataList);
    when(liabilityServiceHelper.mapToObject(Mockito.any())).thenReturn(null); // This simulates failed mapping

    LiabilityDetailModel result = service.getLiabilityDetails(claimId, any());
    assertNotNull(result);
    verify(liabilityDetailModelBuilder, times(0)).build(any(), any(), any(), any());
}
