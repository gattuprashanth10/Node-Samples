public LiabilityDetailModel getLiabilityDetails(String claimId, String negligenceRule) {
    List<LiabilityData> liabilityData = liabilityRepository.getAllByClaimId(claimId);
    String status = liabilityServiceHelper.getStatus(liabilityData);
    String statusCode = liabilityServiceHelper.getStatusCode(status);

    Map<String, List<LiabilityParticipantDetailModel>> groupedDetails = liabilityData.stream()
        .map(liabilityServiceHelper::mapToObject)
        .collect(Collectors.groupingBy(liabilityServiceHelper::getGroupKey));

    Map<String, LiabilityParticipantDetailModel> liabilityModelsMap = new HashMap<>();

    for (List<LiabilityParticipantDetailModel> group : groupedDetails.values()) {
        List<String> primaryParticipantRoles = group.stream()
            .map(data -> data.getPrimaryParticipantRole().get(0))
            .filter(Objects::nonNull)
            .distinct()
            .collect(Collectors.toList());

        List<String> affectedParticipantRoles = group.stream()
            .map(data -> data.getLiabilityDetails().get(0).getAffectedParticipantRole().get(0))
            .filter(Objects::nonNull)
            .distinct()
            .collect(Collectors.toList());

        String adjusterLow = group.stream()
            .map(data -> data.getLiabilityDetails().get(0).getAdjusterLow())
            .filter(Objects::nonNull)
            .distinct()
            .collect(Collectors.joining(""));

        String finalLiabilityPct = group.stream()
            .map(data -> data.getLiabilityDetails().get(0).getFinalLiabilityPct())
            .filter(Objects::nonNull)
            .distinct()
            .collect(Collectors.joining(""));

        String finalLiability = group.stream()
            .map(data -> data.getLiabilityDetails().get(0).getFinalLiability())
            .filter(Objects::nonNull)
            .distinct()
            .collect(Collectors.joining(""));

        String financialLiability = getFinancialLiability(statusCode, negligenceRule, adjusterLow, finalLiabilityPct, primaryParticipantRoles, affectedParticipantRoles);

        AffectedParticipantDetailModel affectedParticipantDetailModel = AffectedParticipantDetailModel.builder()
            .affectedParticipantRole(
                affectedParticipantRoles.stream()
                    .map(role -> codeDecodeHelper.getCodeDecodeShortDesc(PARTICIPANT_ROLE_CATEGORY, role))
                    .collect(Collectors.toList())
            )
            .affectedParticipant(group.get(0).getLiabilityDetails().get(0).getAffectedParticipant())
            .finalLiability(finalLiability)
            .affectedParticipantId(group.get(0).getLiabilityDetails().get(0).getAffectedParticipantId())
            .adjusterLow(group.get(0).getLiabilityDetails().get(0).getAdjusterLow())
            .adjusterHigh(group.get(0).getLiabilityDetails().get(0).getAdjusterHigh())
            .financialLiability(financialLiability)
            .isLiabilityModified(!StringUtils.isEmpty(financialLiability) && isLiabilityModified(statusCode, adjusterLow, finalLiabilityPct, financialLiability))
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
