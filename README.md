@Test
void testCheckValidationBeforeAsyncC_StatusZero_HitCountLessThanLimit() {
    CreateGroupsResponse response = new CreateGroupsResponse();
    RequestHeader header = new RequestHeader();
    header.setCorrelationId("corr123");

    ClientOnboardingRequestTrackEntity trackEntity = new ClientOnboardingRequestTrackEntity();
    trackEntity.setStatus(0);
    trackEntity.setHitCount(1);

    when(configBean.getHitCount()).thenReturn(3);
    when(trackRepo.findByCorrelationIdAndClientId(any(), any())).thenReturn(trackEntity);

    try (MockedStatic<StartAnywhereSecurityUtil> secUtil = mockStatic(StartAnywhereSecurityUtil.class);
         MockedStatic<RequestHeader> reqHeader = mockStatic(RequestHeader.class);
         MockedStatic<StartAnywhereUtil> util = mockStatic(StartAnywhereUtil.class)) {

        secUtil.when(() -> StartAnywhereSecurityUtil.unCleanIt(any())).thenReturn("{}");
        reqHeader.when(() -> RequestHeader.parse(any())).thenReturn(header);
        util.when(() -> StartAnywhereUtil.buildResponse(any(), any(), any(), any(), any(), any(), any(), any()))
            .thenReturn(response);

        CreateGroupsResponse result = service.checkValidationBeforeAsyncC("token", "{}", "client123", "ClientX");

        assertNotNull(result);
        verify(trackRepo).save(trackEntity); // ✅ hitCount should be incremented
        assertEquals(2, trackEntity.getHitCount()); // ✅ check side effect
    }
}

@Test
void testCheckValidationBeforeAsyncC_StatusZero_HitCountExceedsLimit() {
    CreateGroupsResponse response = new CreateGroupsResponse();
    RequestHeader header = new RequestHeader();
    header.setCorrelationId("corr123");

    ClientOnboardingRequestTrackEntity trackEntity = new ClientOnboardingRequestTrackEntity();
    trackEntity.setStatus(0);
    trackEntity.setHitCount(3); // equal to limit → should trigger rate limit response

    when(configBean.getHitCount()).thenReturn(3);
    when(trackRepo.findByCorrelationIdAndClientId(any(), any())).thenReturn(trackEntity);

    try (MockedStatic<StartAnywhereSecurityUtil> secUtil = mockStatic(StartAnywhereSecurityUtil.class);
         MockedStatic<RequestHeader> reqHeader = mockStatic(RequestHeader.class);
         MockedStatic<StartAnywhereUtil> util = mockStatic(StartAnywhereUtil.class)) {

        secUtil.when(() -> StartAnywhereSecurityUtil.unCleanIt(any())).thenReturn("{}");
        reqHeader.when(() -> RequestHeader.parse(any())).thenReturn(header);
        util.when(() -> StartAnywhereUtil.buildResponse(any(), any(), any(), any(), any(), any(), any(), any()))
            .thenReturn(response);

        CreateGroupsResponse result = service.checkValidationBeforeAsyncC("token", "{}", "client123", "ClientX");

        assertNotNull(result);
        verify(trackRepo, never()).save(any()); // ✅ should NOT increment/save when limit exceeded
    }
}

@Test
void testCheckValidationBeforeAsyncC_StatusNotZero_ReturnsCompletedResponse() {
    CreateGroupsResponse response = new CreateGroupsResponse();
    RequestHeader header = new RequestHeader();
    header.setCorrelationId("corr123");

    ClientOnboardingRequestTrackEntity trackEntity = new ClientOnboardingRequestTrackEntity();
    trackEntity.setStatus(1); // completed

    when(trackRepo.findByCorrelationIdAndClientId(any(), any())).thenReturn(trackEntity);

    try (MockedStatic<StartAnywhereSecurityUtil> secUtil = mockStatic(StartAnywhereSecurityUtil.class);
         MockedStatic<RequestHeader> reqHeader = mockStatic(RequestHeader.class);
         MockedStatic<StartAnywhereUtil> util = mockStatic(StartAnywhereUtil.class)) {

        secUtil.when(() -> StartAnywhereSecurityUtil.unCleanIt(any())).thenReturn("{}");
        reqHeader.when(() -> RequestHeader.parse(any())).thenReturn(header);
        util.when(() -> StartAnywhereUtil.buildResponse(any(), any(), any(), any(), any(), any(), any(), any()))
            .thenReturn(response);

        CreateGroupsResponse result = service.checkValidationBeforeAsyncC("token", "{}", "client123", "ClientX");

        assertNotNull(result);
        verify(trackRepo, never()).save(any());
    }
}

@Test
void testCheckValidationBeforeAsyncC_NoTrackEntityFound() {
    CreateGroupsResponse response = new CreateGroupsResponse();
    RequestHeader header = new RequestHeader();
    header.setCorrelationId("corr123");

    when(trackRepo.findByCorrelationIdAndClientId(any(), any())).thenReturn(null);

    try (MockedStatic<StartAnywhereSecurityUtil> secUtil = mockStatic(StartAnywhereSecurityUtil.class);
         MockedStatic<RequestHeader> reqHeader = mockStatic(RequestHeader.class);
         MockedStatic<StartAnywhereUtil> util = mockStatic(StartAnywhereUtil.class)) {

        secUtil.when(() -> StartAnywhereSecurityUtil.unCleanIt(any())).thenReturn("{}");
        reqHeader.when(() -> RequestHeader.parse(any())).thenReturn(header);
        util.when(() -> StartAnywhereUtil.buildResponse(any(), any(), any(), any(), any(), any(), any(), any()))
            .thenReturn(response);

        CreateGroupsResponse result = service.checkValidationBeforeAsyncC("token", "{}", "client123", "ClientX");

        assertNotNull(result);
        verify(trackRepo, never()).save(any());
    }
}
