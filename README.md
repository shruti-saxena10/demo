package com.alight.cc.startanywhere.exception;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;

import java.io.IOException;

import org.junit.jupiter.api.Test;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.http.converter.HttpMessageNotReadableException;

import com.alight.cc.startanywhere.model.BaseResponse;
import com.alight.cc.startanywhere.util.StartAnyWhereConstants;
import com.fasterxml.jackson.databind.JsonMappingException;

class GlobalExceptionHandlerHandleJsonParseErrorTest {

    private final GlobalExceptionHandler handler = new GlobalExceptionHandler();

    /**
     * ✅ Test case: When JsonMappingException contains a field path,
     * ensure the field name is included in the error message.
     */
    @Test
    void handleJsonParseError_withFieldName_returnsSpecificMessage() {
        // Arrange
        JsonMappingException jme = JsonMappingException.fromUnexpectedIOE(new IOException("Test"));
        jme.prependPath(new JsonMappingException.Reference(Object.class, "age")); // simulate field path
        HttpMessageNotReadableException ex = new HttpMessageNotReadableException("JSON parse error", jme);

        // Act
        ResponseEntity<Object> response = handler.handleJsonParseError(ex);

        // Assert
        assertEquals(HttpStatus.BAD_REQUEST, response.getStatusCode());
        BaseResponse body = (BaseResponse) response.getBody();
        assertEquals(StartAnyWhereConstants.HTTP_STATUS_BAD_REQUEST, body.getResponseCode());
        assertTrue(body.getResponseDescription().contains("Invalid value for field 'age'"));
    }

    /**
     * ✅ Test case: When JsonMappingException exists but has no path,
     * ensure the generic invalid input message is used.
     */
    @Test
    void handleJsonParseError_withoutFieldName_returnsGenericMessage() {
        // Arrange
        JsonMappingException jme = JsonMappingException.fromUnexpectedIOE(new IOException("Test"));
        HttpMessageNotReadableException ex = new HttpMessageNotReadableException("JSON parse error", jme);

        // Act
        ResponseEntity<Object> response = handler.handleJsonParseError(ex);

        // Assert
        assertEquals(HttpStatus.BAD_REQUEST, response.getStatusCode());
        BaseResponse body = (BaseResponse) response.getBody();
        assertEquals(StartAnyWhereConstants.HTTP_STATUS_BAD_REQUEST, body.getResponseCode());
        assertEquals("Invalid input provided. Please check your request payload.", body.getResponseDescription());
    }

    /**
     * ✅ Test case: When cause is not a JsonMappingException,
     * ensure it falls back to the generic error message.
     */
    @Test
    void handleJsonParseError_withNonJsonMappingCause_returnsGenericMessage() {
        // Arrange
        Throwable cause = new RuntimeException("Some other parsing exception");
        HttpMessageNotReadableException ex = new HttpMessageNotReadableException("JSON parse error", cause);

        // Act
        ResponseEntity<Object> response = handler.handleJsonParseError(ex);

        // Assert
        assertEquals(HttpStatus.BAD_REQUEST, response.getStatusCode());
        BaseResponse body = (BaseResponse) response.getBody();
        assertEquals(StartAnyWhereConstants.HTTP_STATUS_BAD_REQUEST, body.getResponseCode());
        assertEquals("Invalid input provided. Please check your request payload.", body.getResponseDescription());
    }

    /**
     * ✅ Test case: When there is no cause at all,
     * ensure the generic invalid input message is still returned.
     */
    @Test
    void handleJsonParseError_withoutCause_returnsGenericMessage() {
        // Arrange
        HttpMessageNotReadableException ex = new HttpMessageNotReadableException("JSON parse error", (Throwable) null);

        // Act
        ResponseEntity<Object> response = handler.handleJsonParseError(ex);

        // Assert
        assertEquals(HttpStatus.BAD_REQUEST, response.getStatusCode());
        BaseResponse body = (BaseResponse) response.getBody();
        assertEquals(StartAnyWhereConstants.HTTP_STATUS_BAD_REQUEST, body.getResponseCode());
        assertEquals("Invalid input provided. Please check your request payload.", body.getResponseDescription());
    }
}
