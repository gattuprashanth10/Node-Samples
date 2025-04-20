@Test
void testGetLiabilityDetails_fullCoverage() {
    // Arrange
    LiabilityParticipantDetailModel mockParticipant = mock(LiabilityParticipantDetailModel.class);
    LiabilityDetailModel.LiabilityDetails mockLiabilityDetails = mock(LiabilityDetailModel.LiabilityDetails.class);
    
    List<LiabilityParticipantDetailModel> group = List.of(mockParticipant);
    String statusCode = "SomeStatus";
    
    // Mock inner LiabilityDetails
    when(mockParticipant.getLiabilityDetails()).thenReturn(List.of(mockLiabilityDetails));
    
    // PrimaryParticipantRole
    when(mockParticipant.getPrimaryParticipantRole()).thenReturn(List.of("ROLE_1"));

    // LiabilityDetail fields
    when(mockLiabilityDetails.getAffectedParticipantRole()).thenReturn(List.of("AFFECTED_ROLE"));
    when(mockLiabilityDetails.getAdjusterLow()).thenReturn("Low1");
    when(mockLiabilityDetails.getFinalLiabilityPct()).thenReturn("50");
    when(mockLiabilityDetails.getFinancialLiability()).thenReturn("5000");

    // Helpers - distinct list
    when(liabilityServiceHelper.joinNonNullDistinctList(eq(group), any()))
        .thenAnswer(invocation -> {
            // Actually call the lambda to ensure it's covered
            List<LiabilityParticipantDetailModel> list = invocation.getArgument(0);
            Function<LiabilityParticipantDetailModel, String> extractor = invocation.getArgument(1);
            return list.stream().map(extractor).distinct().collect(Collectors.toList());
        });

    // Helpers - distinct strings
    when(liabilityServiceHelper.joinNonNullDistinctStrings(eq(group), any(), eq("")))
        .thenAnswer(invocation -> {
            Function<LiabilityParticipantDetailModel, String> extractor = invocation.getArgument(1);
            return group.stream().map(extractor).filter(Objects::nonNull).distinct().collect(Collectors.joining(","));
        });

    // Helper - isLiabilityModified
    when(liabilityServiceHelper.isLiabilityModified(eq(statusCode), eq("Low1"), eq("50"), eq("5000")))
        .thenReturn(true);

    // CodeDecode
    when(codeDecodeHelper.getCodeDecodeShortDesc(PARTICIPANT_ROLE_CATEGORY, "ROLE_1"))
        .thenReturn("Primary Role 1");

    // Act
    LiabilityDetails liabilityDetails = yourClassUnderTest.getLiabilityDetails(group, statusCode); // Replace with your actual method call

    // Assert
    assertNotNull(liabilityDetails);
    assertEquals(List.of("Primary Role 1"), liabilityDetails.getPrimaryParticipantRole());
    assertTrue(liabilityDetails.isLiabilityModified());
}
