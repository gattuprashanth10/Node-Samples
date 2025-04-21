private LiabilityParticipantDetailModel createMockLiabilityParticipantDetailModelWithDetails() {
        LiabilityParticipantDetailModel mockModel = Mockito.mock(LiabilityParticipantDetailModel.class);
        List<AffectedParticipantDetailModel> liabilityDetailsList = new ArrayList<>();
        AffectedParticipantDetailModel mockAffectedDetail = Mockito.mock(AffectedParticipantDetailModel.class);

        when(mockAffectedDetail.getAffectedParticipant()).thenReturn("Mock Affected Participant");
        when(mockAffectedDetail.getAffectedParticipantId()).thenReturn("MockAffectedId");
        when(mockAffectedDetail.getFinalLiability()).thenReturn("Default Final");
        when(mockAffectedDetail.getAdjusterLow()).thenReturn("Default Low");
        when(mockAffectedDetail.getAdjusterHigh()).thenReturn("Default High");

        liabilityDetailsList.add(mockAffectedDetail);
        when(mockModel.getLiabilityDetails()).thenReturn(liabilityDetailsList);
        when(mockModel.getPrimaryParticipantId()).thenReturn("MockPrimaryId");
        when(mockModel.getPrimaryParticipant()).thenReturn("Mock Primary Participant");

        return mockModel;
    }

    @Test
    void testGetLiabilityDetails_FinancialLiabilityIsEmpty_CoversBranch() {

        when(liabilityRepository.getAllByClaimId(claimId)).thenReturn(Collections.singletonList(new LiabilityData()));

        when(liabilityServiceHelper.mapToObject(any())).thenReturn(createMockLiabilityParticipantDetailModelWithDetails());
        when(liabilityServiceHelper.getGroupKey(any())).thenReturn("groupKey");
        when(liabilityServiceHelper.joinNonNullDistinctList(anyList(), any())).thenReturn(Arrays.asList("Role1", "Role2"));
        when(codeDecodeHelper.getCodeDecodeShortDesc(any(), anyString())).thenReturn("Role Description");


        when(service.getFinancialLiability(any(), any(), any(), any(), any(), any())).thenReturn("");

        when(liabilityServiceHelper.getStatusCode(any())).thenReturn(INITIAL);

        when(liabilityServiceHelper.joinNonNullDistinctStrings(anyList(), any(), eq(EMPTY_STRING)))
             .thenReturn("AdjusterLowValue")
             .thenReturn("FinalPctValue");

        when(liabilityDetailModelBuilder.build(any(), any(), any(), any())).thenReturn(new LiabilityDetailModel());

        service.getLiabilityDetails(claimId, negligenceRule);

        verify(service, never()).isLiabilityModified(any(), any(), any(), any());
    }

    @Test
    void testGetLiabilityDetails_FinancialLiabilityNotEmptyAndIsNotModified_CoversBranch() {

        String financialLiabilityValue = "50";
        String adjusterLowValue = "50";
        String finalLiabilityPctValue = "75";
        String statusCode = INITIAL;

        when(liabilityRepository.getAllByClaimId(claimId)).thenReturn(Collections.singletonList(new LiabilityData()));
        when(liabilityServiceHelper.mapToObject(any())).thenReturn(createMockLiabilityParticipantDetailModelWithDetails());
        when(liabilityServiceHelper.getGroupKey(any())).thenReturn("groupKey");
        when(liabilityServiceHelper.joinNonNullDistinctList(anyList(), any())).thenReturn(Arrays.asList("Role1", "Role2"));
         when(codeDecodeHelper.getCodeDecodeShortDesc(any(), anyString())).thenReturn("Role Description");

        when(service.getFinancialLiability(any(), any(), any(), any(), any(), any())).thenReturn(financialLiabilityValue);

        when(liabilityServiceHelper.getStatusCode(any())).thenReturn(statusCode);

        when(liabilityServiceHelper.joinNonNullDistinctStrings(anyList(), any(), eq(EMPTY_STRING)))
                .thenReturn(adjusterLowValue)
                .thenReturn(finalLiabilityPctValue);

        when(liabilityDetailModelBuilder.build(any(), any(), any(), any())).thenReturn(new LiabilityDetailModel());

        service.getLiabilityDetails(claimId, negligenceRule);

        verify(service, times(1)).isLiabilityModified(eq(statusCode), eq(adjusterLowValue), eq(finalLiabilityPctValue), eq(financialLiabilityValue));
    }

    @Test
    void testGetLiabilityDetails_FinancialLiabilityNotEmptyAndIsModified_CoversBranch() {

        String financialLiabilityValue = "50";
        String adjusterLowValue = "60";
        String finalLiabilityPctValue = "75";
        String statusCode = INITIAL;

        when(liabilityRepository.getAllByClaimId(claimId)).thenReturn(Collections.singletonList(new LiabilityData()));
        when(liabilityServiceHelper.mapToObject(any())).thenReturn(createMockLiabilityParticipantDetailModelWithDetails());
        when(liabilityServiceHelper.getGroupKey(any())).thenReturn("groupKey");
        when(liabilityServiceHelper.joinNonNullDistinctList(anyList(), any())).thenReturn(Arrays.asList("Role1", "Role2"));
        when(codeDecodeHelper.getCodeDecodeShortDesc(any(), anyString())).thenReturn("Role Description");

        when(service.getFinancialLiability(any(), any(), any(), any(), any(), any())).thenReturn(financialLiabilityValue);

        when(liabilityServiceHelper.getStatusCode(any())).thenReturn(statusCode);

        when(liabilityServiceHelper.joinNonNullDistinctStrings(anyList(), any(), eq(EMPTY_STRING)))
                .thenReturn(adjusterLowValue)
                .thenReturn(finalLiabilityPctValue);

        when(liabilityDetailModelBuilder.build(any(), any(), any(), any())).thenReturn(new LiabilityDetailModel());

        service.getLiabilityDetails(claimId, negligenceRule);

        verify(service, times(1)).isLiabilityModified(eq(statusCode), eq(adjusterLowValue), eq(finalLiabilityPctValue), eq(financialLiabilityValue));
    }
