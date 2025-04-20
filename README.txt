public LiabilityDetailModel getLiabilityDetails(String claimId, String negligenceRule) {
    List<LiabilityData> liabilityData = liabilityRepository.getAllByClaimId(claimId);
    String status = liabilityServiceHelper.getStatus(liabilityData);
    String statusCode = liabilityServiceHelper.getStatusCode(status);

    Map<String, List<LiabilityParticipantDetailModel>> groupedDetails = liabilityData.stream()
        .map(liabilityServiceHelper::mapToObject)
        .collect(Collectors.groupingBy(liabilityServiceHelper::getGroupKey));

    Map<String, LiabilityParticipantDetailModel> liabilityModelsMap = new HashMap<>();

    for (List<LiabilityParticipantDetailModel> group : groupedDetails.values()) {
        if (group == null || group.isEmpty()) continue;

        List<String> primaryParticipantRoles = group.stream()
            .map(data -> safeGet(data.getPrimaryParticipantRole(), 0))
            .filter(Objects::nonNull)
            .distinct()
            .collect(Collectors.toList());

        List<String> affectedParticipantRoles = group.stream()
            .map(data -> safeGet(data.getLiabilityDetails(), 0))
            .map(detail -> safeGet(detail.getAffectedParticipantRole(), 0))
            .filter(Objects::nonNull)
            .distinct()
            .collect(Collectors.toList());

        String adjusterLow = group.stream()
            .map(data -> safeGet(data.getLiabilityDetails(), 0))
            .map(LiabilityDetail::getAdjusterLow)
            .filter(Objects::nonNull)
            .distinct()
            .collect(Collectors.joining(""));

        String finalLiabilityPct = group.stream()
            .map(data -> safeGet(data.getLiabilityDetails(), 0))
            .map(LiabilityDetail::getFinalLiabilityPct)
            .filter(Objects::nonNull)
            .distinct()
            .collect(Collectors.joining(""));

        String finalLiability = group.stream()
            .map(data -> safeGet(data.getLiabilityDetails(), 0))
            .map(LiabilityDetail::getFinalLiability)
            .filter(Objects::nonNull)
            .distinct()
            .collect(Collectors.joining(""));

        String financialLiability = getFinancialLiability(
            statusCode, negligenceRule, adjusterLow, finalLiabilityPct,
            primaryParticipantRoles, affectedParticipantRoles
        );

        LiabilityDetail firstDetail = safeGet(group.get(0).getLiabilityDetails(), 0);

        AffectedParticipantDetailModel affectedParticipantDetailModel = AffectedParticipantDetailModel.builder()
            .affectedParticipantRole(affectedParticipantRoles.stream()
                .map(role -> codeDecodeHelper.getCodeDecodeShortDesc(PARTICIPANT_ROLE_CATEGORY, role))
                .collect(Collectors.toList()))
            .affectedParticipant(firstDetail != null ? firstDetail.getAffectedParticipant() : null)
            .finalLiability(finalLiability)
            .affectedParticipantId(firstDetail != null ? firstDetail.getAffectedParticipantId() : null)
            .adjusterLow(firstDetail != null ? firstDetail.getAdjusterLow() : null)
            .adjusterHigh(firstDetail != null ? firstDetail.getAdjusterHigh() : null)
            .financialLiability(financialLiability)
            .isLiabilityModified(!StringUtils.isEmpty(financialLiability) &&
                isLiabilityModified(statusCode, adjusterLow, finalLiabilityPct, financialLiability))
            .build();

        String primaryParticipantId = group.get(0).getPrimaryParticipantId();

        if (liabilityModelsMap.containsKey(primaryParticipantId)) {
            liabilityModelsMap.get(primaryParticipantId).getLiabilityDetails().add(affectedParticipantDetailModel);
        } else {
            List<AffectedParticipantDetailModel> affectedParticipantDetailModels = new ArrayList<>();
            affectedParticipantDetailModels.add(affectedParticipantDetailModel);
            liabilityModelsMap.put(primaryParticipantId,
                LiabilityParticipantDetailModel.builder()
                    .primaryParticipant(group.get(0).getPrimaryParticipant())
                    .liabilityDetails(affectedParticipantDetailModels)
                    .primaryParticipantRole(
                        primaryParticipantRoles.stream()
                            .map(role -> codeDecodeHelper.getCodeDecodeShortDesc(PARTICIPANT_ROLE_CATEGORY, role))
                            .collect(Collectors.toList())
                    )
                    .primaryParticipantId(primaryParticipantId)
                    .build()
            );
        }
    }

    List<LiabilityParticipantDetailModel> liabilityModels = new ArrayList<>(liabilityModelsMap.values());
    return liabilityDetailModelBuilder.build(status, statusCode, liabilityModels, negligenceRule);
}

// Helper to safely get an element from a list
private <T> T safeGet(List<T> list, int index) {
    return (list != null && list.size() > index) ? list.get(index) : null;
}
