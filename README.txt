@Test
public void testGetLiabilityDetails_fullCoverage() {
    List<LiabilityData> liabilityDataList = Arrays.asList(new LiabilityData(), new LiabilityData());
    
    // Simulate detailed participant model
    LiabilityParticipantDetailModel participant1 = getLiablilityParticipantDetails("123", "test", "affected1");
    participant1.getLiabilityDetails().add(getAffectedParticipantDetailModel("extra1"));
    
    LiabilityParticipantDetailModel participant2 = getLiablilityParticipantDetails("456", "test2", "affected2");

    // Mock repository and helpers
    when(liabilityRepository.getAllByClaimId(claimId)).thenReturn(liabilityDataList);
    when(liabilityServiceHelper.mapToObject(any()))
        .thenReturn(participant1)
        .thenReturn(participant2);
    when(liabilityServiceHelper.getGroupKey(any()))
        .thenReturn("Key1")
        .thenReturn("Key2");
    when(liabilityServiceHelper.joinNonNullDistinctStrings(any(), any(), any())).thenReturn("Roles");
    
    LiabilityDetailModel mockModel = new LiabilityDetailModel(); // or build a detailed one
    when(liabilityDetailModelBuilder.build(any(), any(), any(), any())).thenReturn(mockModel);
    
    // Act
    LiabilityDetailModel result = service.getLiabilityDetails(claimId, null);
    
    // Assert
    assertNotNull(result);
    verify(liabilityDetailModelBuilder, times(1)).build(
        any(), any(),
        any(),
        any()
    );
}
