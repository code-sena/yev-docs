# ADR-006 — Wompi Payment Gateway Integration

| Field | Value |
|-------|-------|
| **ID** | ADR-006 |
| **Date** | 2024-09-08 |
| **Status** | Accepted |
| **Authors** | Cristofer Lozano — Tech Lead, Jhampier Santos — DevOps |
| **Reviewers** | Development Team |

---

## Context

TuEvento requires a secure, reliable payment processing solution for the Colombian market that supports:

- **Multiple payment methods**: Credit/debit cards, PSE (bank transfers), Nequi, cash payments
- **Colombian market compliance**: Local regulations and tax requirements
- **Security standards**: PCI DSS compliance for card processing
- **Real-time processing**: Immediate payment confirmation for seat reservations
- **Mobile optimization**: Smooth payment experience on mobile devices
- **Webhook notifications**: Real-time payment status updates
- **Refund capabilities**: Support for event cancellations and refunds
- **Competitive rates**: Reasonable processing fees for small/medium events

The payment gateway must integrate seamlessly with the seat reservation system to prevent double bookings.

---

## Evaluated alternatives

### Alternative A — Wompi (CHOSEN)
**Colombian payment processor with comprehensive local support**
- **Pros:**
  - **Colombian market leader**: Deep understanding of local payment preferences
  - **Multiple payment methods**: Cards, PSE, Nequi, Daviplata, cash (Efecty, Baloto)
  - **Competitive rates**: Lower fees than international providers for Colombian market
  - **PCI DSS compliant**: Handles sensitive card data processing
  - **Excellent documentation**: Well-documented REST API with SDKs
  - **Mobile-optimized**: Native mobile payment flows
  - **Real-time webhooks**: Immediate payment status notifications
  - **Tokenization**: Secure card tokenization for recurring payments
  - **Colombian peso native**: No currency conversion issues
  - **Local support**: Spanish-speaking support team in Colombian timezone
- **Cons:**
  - **Geographic limitation**: Primarily focused on Colombian market
  - **Smaller ecosystem**: Less third-party tooling compared to global providers
  - **API rate limits**: More restrictive than larger international providers

### Alternative B — PayU Colombia
**International payment provider with Colombian presence**
- **Pros:**
  - **International backing**: Part of larger Prosus/Naspers group
  - **Multiple countries**: Experience across Latin America
  - **Comprehensive features**: Advanced fraud detection, recurring payments
  - **Enterprise features**: Better reporting and analytics tools
- **Cons:**
  - **Higher costs**: More expensive fees, especially for smaller transactions
  - **Complex integration**: More complex API with steeper learning curve
  - **Limited local methods**: Weaker support for local payment methods (Nequi, Daviplata)
  - **International focus**: Less optimization for Colombian market specifics

### Alternative C — Stripe
**Global payment leader with international expansion**
- **Pros:**
  - **Excellent developer experience**: Outstanding documentation and SDKs
  - **Rich ecosystem**: Extensive third-party integrations and tools
  - **Advanced features**: Sophisticated fraud detection, payment optimization
  - **Global reach**: Easy expansion to other markets
- **Cons:**
  - **Limited Colombian support**: No PSE, Nequi, or cash payment support
  - **Higher costs**: International processing fees
  - **Currency conversion**: USD/COP conversion adds complexity and fees
  - **Compliance**: Additional complexity for Colombian tax reporting

### Alternative D — MercadoPago Colombia
**Latin American payment provider owned by MercadoLibre**
- **Pros:**
  - **Regional expertise**: Strong presence in Latin America
  - **Local payment methods**: Good support for Colombian payment preferences
  - **E-commerce integration**: Strong integration with online marketplaces
  - **Brand recognition**: Well-known brand in Colombian market
- **Cons:**
  - **Marketplace focus**: Optimized for marketplace transactions, not events
  - **Limited customization**: Less flexible for custom payment flows
  - **Documentation quality**: Less comprehensive technical documentation
  - **Support quality**: Mixed reviews on technical support responsiveness

### Alternative E — Custom Payment Processing
**Direct integration with Colombian banks and payment networks**
- **Pros:**
  - **Maximum control**: Complete control over payment flow and user experience
  - **Lower long-term costs**: No payment processor fees
  - **Custom features**: Tailored functionality for specific business needs
- **Cons:**
  - **Massive complexity**: Requires PCI DSS certification, bank integrations
  - **Development time**: Months of development vs. weeks with existing processor
  - **Compliance burden**: Direct responsibility for security and regulatory compliance
  - **Maintenance overhead**: Ongoing maintenance of complex payment infrastructure
  - **Risk**: High risk of security vulnerabilities and compliance violations

---

## Decision

**Alternative A:** Wompi Payment Gateway Integration

**Integration architecture:**
```
TuEvento → Wompi API → Colombian Payment Networks
    ↓
Wompi Webhooks → TuEvento (payment status updates)
```

**Supported payment methods:**
- **Credit/Debit Cards**: Visa, Mastercard, American Express, Diners Club
- **PSE (Pagos Seguros en Línea)**: Direct bank transfers via ACH Colombia
- **Digital wallets**: Nequi, Daviplata
- **Cash payments**: Efecty, Baloto, Su Red network
- **International cards**: For international attendees

**Key integration features:**
- **Tokenization**: Secure storage of payment methods for future use
- **3D Secure**: Additional security layer for card payments
- **Real-time webhooks**: Immediate payment status notifications
- **Refund API**: Automated refund processing for event cancellations
- **Fraud protection**: Built-in fraud detection and prevention

**Technical implementation:**
```java
@Service
public class WompiPaymentService {
    
    public PaymentResponse processPayment(PaymentRequest request) {
        // 1. Create payment intent with Wompi
        WompiPaymentIntent intent = wompiApiClient.createPaymentIntent(
            WompiPaymentIntentRequest.builder()
                .amountInCents(request.getAmountInCents())
                .currency("COP")
                .reference(request.getOrderId())
                .customerEmail(request.getCustomerEmail())
                .redirectUrl(request.getRedirectUrl())
                .build()
        );
        
        // 2. Store payment record
        Payment payment = Payment.builder()
            .orderId(request.getOrderId())
            .wompiPaymentIntentId(intent.getId())
            .status(PaymentStatus.PENDING)
            .amount(request.getAmountInCents())
            .currency("COP")
            .build();
        paymentRepository.save(payment);
        
        // 3. Return payment URL for user redirect
        return PaymentResponse.builder()
            .paymentUrl(intent.getPaymentUrl())
            .paymentIntentId(intent.getId())
            .build();
    }
    
    @EventListener
    public void handleWompiWebhook(WompiWebhookEvent event) {
        // Verify webhook signature
        if (!wompiWebhookValidator.isValidSignature(event)) {
            log.warn("Invalid Wompi webhook signature");
            return;
        }
        
        // Process payment status update
        Payment payment = paymentRepository
            .findByWompiPaymentIntentId(event.getPaymentIntentId())
            .orElseThrow(() -> new PaymentNotFoundException("Payment not found"));
            
        payment.updateStatus(event.getStatus());
        payment.setWompiTransactionId(event.getTransactionId());
        paymentRepository.save(payment);
        
        // Publish domain event for other modules
        eventPublisher.publishEvent(new PaymentStatusUpdatedEvent(
            payment.getId(), payment.getStatus(), payment.getOrderId()
        ));
    }
}
```

---

## Consequences

**Positive:**
- **Local market fit**: Perfect alignment with Colombian payment preferences and regulations
- **Cost efficiency**: Competitive rates for local transactions, especially smaller amounts typical of event tickets
- **User experience**: Familiar payment methods increase conversion rates
- **Compliance**: PCI DSS compliance handled by Wompi, reducing our compliance burden
- **Development velocity**: Well-documented API enables rapid integration
- **Real-time updates**: Webhook notifications provide immediate payment confirmation
- **Mobile optimization**: Native mobile payment flows improve user experience
- **Multi-method support**: Comprehensive coverage of Colombian payment landscape

**Negative:**
- **Vendor lock-in**: Tight coupling with Wompi-specific API and features
- **Geographic limitation**: Difficult expansion to other countries without additional integrations
- **API limitations**: Rate limiting and feature constraints compared to global providers
- **Single point of failure**: Dependency on Wompi's service availability

**Risk mitigation strategies:**
- **Graceful degradation**: Handle Wompi service outages with appropriate user messaging
- **Monitoring**: Comprehensive monitoring of Wompi API response times and error rates
- **Backup communication**: Alternative channels for payment status in case of webhook failures
- **Testing**: Comprehensive testing with Wompi sandbox environment
- **Documentation**: Maintain detailed integration documentation for future modifications

**Financial impact:**
- **Processing fees**: 3.4% + $700 COP for card transactions, lower for PSE and digital wallets
- **Monthly costs**: No monthly fees, pay-per-transaction model
- **Development costs**: Estimated 2-3 weeks of development time vs. 3-6 months for custom solution
- **Compliance costs**: Avoided PCI DSS certification costs (~$50,000+ annually)

**Integration phases:**
- **Phase 1**: Basic card payment processing with webhook handling
- **Phase 2**: PSE and digital wallet integration (Nequi, Daviplata)
- **Phase 3**: Cash payment methods (Efecty, Baloto) 
- **Phase 4**: Advanced features (tokenization, recurring payments)

**Security considerations:**
- **No card data storage**: All sensitive payment data handled by Wompi
- **Webhook validation**: Cryptographic signature verification for all webhooks
- **HTTPS enforcement**: All Wompi API communication over secure connections
- **PCI DSS scope**: Reduced PCI DSS scope due to external payment processing
- **Audit logging**: Complete audit trail for all payment operations

---

## References

- Payment module implementation → `modules/payment/`
- Wompi API documentation → https://docs.wompi.co/
- Payment security requirements → `evidence/security-threat-model.md#payment-information-disclosure`
- Order processing flow → `modules/ticket/application/usecase/ProcessOrderUseCase.java`
- Webhook handling → `modules/payment/infrastructure/external/WompiWebhookController.java`
- Payment audit logging → `evidence/cross-cutting.md#audit-logging`