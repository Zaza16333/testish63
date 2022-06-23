
package main

import (
	"context"
	swish "github.com/Kansuler/payment-swish"
	"github.com/satori/go.uuid"
	"io/ioutil"
	"log"
)

func main() {
	cert, err := ioutil.ReadFile("certificates/Swish_Merchant_TestCertificate_1234679304.p12")
	if err != nil {
		log.Fatalf("could not load test certificate: %s", err.Error())
	}

	s, err := swish.New(swish.Options{
		Passphrase:     "swish",
		CA:             swish.TestCertificate,
		SSLCertificate: cert,
		Test:           true,
		Timeout:        5,
	})
	if err != nil {
		log.Fatalf("could not create swish instance: %s", err.Error())
	}

	paymentRequest := swish.CreatePaymentRequestOptions{
		InstructionUUID:       uuid.NewV4().String(),
		CallbackURL:           "https://localhost:8080/callback",
		PayeeAlias:            "1234679304",
		Amount:                "100.01",
		Currency:              "SEK",
		PayeePaymentReference: "",
		PayerAlias:            "",
		PayerSSN:              "",
		PayerAgeLimit:         "",
		Message:               "",
	}

	payment, err := s.CreatePaymentRequest(context.Background(), paymentRequest)
	if err != nil {
		log.Fatalf("could not create payment paymentRequest: %s", err.Error())
	}

	_, err = s.Status(context.Background(), payment.Location)
	if err != nil {
		log.Fatalf("could not get status of payment: %s", err.Error())
	}

	refundRequest := swish.CreateRefundOptions{
		InstructionUUID:          uuid.NewV4().String(),
		OriginalPaymentReference: paymentRequest.InstructionUUID,
		CallbackURL:              paymentRequest.CallbackURL,
		PayerAlias:               paymentRequest.PayeeAlias,
		Amount:                   paymentRequest.Amount,
		Currency:                 paymentRequest.Currency,
		PayerPaymentReference:    "123",
		Message:                  "Refund",
	}

	refund, err := s.CreateRefund(context.Background(), refundRequest)
	if err != nil {
		log.Fatalf("could not create refund: %s", err.Error())
	}

	_, err = s.Status(context.Background(), refund.Location)
	if err != nil {
		log.Fatalf("could not get status of refund: %s", err.Error())
	}
}
