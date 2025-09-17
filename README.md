public class EmailSender {

    private static final int MAX_RETRY_COUNT = 3;
    private static final long BASE_RETRY_DELAY_MS = 2000;  // 2 seconds base delay

    private final EmailService emailService;

    public EmailSender(EmailService emailService) {
        this.emailService = emailService;
    }

    public MimeMessage sendEmail(EmailDto emailDto) {
        validateEmails(emailDto);

        for (int attempt = 1; attempt <= MAX_RETRY_COUNT; attempt++) {
            try {
                MimeMessage message = emailService.createMessage();
                MimeMessageHelper messageHelper = emailService.getMimeMessageHelper(emailDto);

                messageHelper.setText(emailDto.getEmailContent(), true);
                emailService.setAttachment(messageHelper, emailDto.getAttachmentList());

                String[] emailRecipients = emailDto.getEmailRecipients().stream()
                    .filter(StringUtils::isNotBlank)
                    .toArray(String[]::new);

                messageHelper.setTo(emailRecipients);

                log.info("Attempt {}: Sending email to {}", attempt, Arrays.toString(emailRecipients));

                emailService.send(message);

                emailDto.setEmailSentStatus(EmailStatus.SUCCESS);
                log.info("Email sent successfully to {}", Arrays.toString(emailRecipients));

                return message;  // SUCCESS

            } catch (MailSendException e) {
                log.error("MailSendException on attempt {}: {}", attempt, e.getMessage());

                // Remove invalid addresses
                Set<String> invalidEmails = getInvalidAddresses(e);
                List<String> remainingRecipients = emailDto.getEmailRecipients().stream()
                    .filter(r -> !invalidEmails.contains(r))
                    .collect(Collectors.toList());

                emailDto.setEmailRecipients(remainingRecipients);

                if (remainingRecipients.isEmpty()) {
                    throw new SendEmailException("All recipients are invalid", e);
                }

                if (attempt == MAX_RETRY_COUNT) {
                    throw new SendEmailException("Max retry limit reached", e);
                }

                // Exponential backoff before retrying
                sleepWithBackoff(attempt);

            } catch (MessagingException | NullPointerException e) {
                log.error("Critical failure sending email: {}", e.getMessage());
                throw new SendEmailException("Non-retriable error", e);
            }
        }

        throw new SendEmailException("Email not sent after retries");
    }

    private void validateEmails(EmailDto emailDto) {
        if (!EmailValidator.getInstance().isValid(emailDto.getSenderEmail())) {
            throw new InvalidEmailException("Invalid sender email: " + emailDto.getSenderEmail());
        }

        List<String> invalidRecipients = emailDto.getEmailRecipients().stream()
            .filter(r -> !EmailValidator.getInstance().isValid(r))
            .collect(Collectors.toList());

        if (!invalidRecipients.isEmpty()) {
            throw new InvalidEmailException("Invalid recipient emails: " + invalidRecipients);
        }
    }

    private Set<String> getInvalidAddresses(MailSendException e) {
        Set<String> invalidEmails = new HashSet<>();
        for (Exception ex : e.getMessageExceptions()) {
            if (ex instanceof SendFailedException sendFailedEx) {
                Address[] invalidAddresses = sendFailedEx.getInvalidAddresses();
                if (invalidAddresses != null) {
                    for (Address addr : invalidAddresses) {
                        invalidEmails.add(addr.toString());
                    }
                }
            }
        }
        return invalidEmails;
    }

    private void sleepWithBackoff(int attempt) {
        try {
            long delay = BASE_RETRY_DELAY_MS * (long) Math.pow(2, attempt - 1);
            log.info("Retrying in {} ms...", delay);
            Thread.sleep(delay);
        } catch (InterruptedException ie) {
            Thread.currentThread().interrupt();
            throw new SendEmailException("Retry interrupted", ie);
        }
    }
}
