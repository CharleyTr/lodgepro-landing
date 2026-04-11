const https = require('https');

function sendEmail(to, subject, html, brevoKey) {
  return new Promise((resolve, reject) => {
    const body = JSON.stringify({
      sender: { name: "LodgePro", email: "c.trigano@gmail.com" },
      to: [{ email: to }],
      subject,
      htmlContent: html,
    });
    const options = {
      hostname: 'api.brevo.com',
      path: '/v3/smtp/email',
      method: 'POST',
      headers: {
        'api-key': brevoKey,
        'Content-Type': 'application/json',
        'Content-Length': Buffer.byteLength(body),
      }
    };
    const req = https.request(options, res => resolve(res.statusCode));
    req.on('error', reject);
    req.write(body);
    req.end();
  });
}

exports.handler = async (event) => {
  if (event.httpMethod !== 'POST') {
    return { statusCode: 405, body: 'Method Not Allowed' };
  }

  try {
    const payload = JSON.parse(event.body);
    const stripeEvent = payload;

    // Vérifier que c'est un paiement réussi
    if (stripeEvent.type !== 'checkout.session.completed' &&
        stripeEvent.type !== 'invoice.payment_succeeded') {
      return { statusCode: 200, body: 'Event ignoré' };
    }

    // Extraire les données du client
    let customerEmail = '';
    let customerName  = '';
    let amount        = 0;
    let formule       = '';

    if (stripeEvent.type === 'checkout.session.completed') {
      const session = stripeEvent.data.object;
      customerEmail = session.customer_details?.email || '';
      customerName  = session.customer_details?.name  || '';
      amount        = (session.amount_total || 0) / 100;
    } else {
      const invoice = stripeEvent.data.object;
      customerEmail = invoice.customer_email || '';
      amount        = (invoice.amount_paid   || 0) / 100;
    }

    // Déterminer la formule selon le montant
    if (amount <= 20)      formule = 'Starter';
    else if (amount <= 40) formule = 'Pro';
    else                   formule = 'Business';

    const BREVO_KEY = process.env.BREVO_API_KEY;
    const ADMIN_EMAIL = 'c.trigano@gmail.com';

    // Email à l'admin
    const htmlAdmin = `
      <div style="font-family:Arial;max-width:600px;margin:auto;border:1px solid #e0e0e0;border-radius:8px;overflow:hidden">
        <div style="background:#1565C0;padding:20px;text-align:center">
          <h2 style="color:white;margin:0">💳 Nouveau paiement LodgePro !</h2>
        </div>
        <div style="padding:24px">
          <p>Un nouveau client vient de s'abonner :</p>
          <div style="background:#F4F7FF;border-radius:8px;padding:16px;margin:16px 0">
            <p><b>📧 Email :</b> ${customerEmail}</p>
            <p><b>👤 Nom :</b> ${customerName || 'Non renseigné'}</p>
            <p><b>💶 Montant :</b> ${amount}€/mois</p>
            <p><b>📦 Formule :</b> ${formule}</p>
          </div>
          <p style="text-align:center;margin-top:24px">
            <a href="https://lodgepro-app-8splavj8ph8xpv2t3avavz.streamlit.app"
               style="background:#1565C0;color:white;padding:14px 28px;border-radius:8px;text-decoration:none;font-weight:bold">
              ➕ Créer l'accès client →
            </a>
          </p>
        </div>
      </div>`;

    // Email de confirmation au client
    const htmlClient = `
      <div style="font-family:Arial;max-width:600px;margin:auto;border:1px solid #e0e0e0;border-radius:8px;overflow:hidden">
        <div style="background:#1565C0;padding:20px;text-align:center">
          <h2 style="color:white;margin:0">🏖️ Merci pour votre abonnement LodgePro !</h2>
        </div>
        <div style="padding:24px;line-height:1.7">
          <p>Bonjour ${customerName || ''},</p>
          <p>Votre abonnement <b>LodgePro ${formule}</b> est confirmé.</p>
          <div style="background:#E8F5E9;border-radius:8px;padding:16px;margin:16px 0">
            <p>🎁 <b>Vos 3 premiers mois sont offerts !</b></p>
            <p>Votre premier prélèvement n'aura lieu que dans 3 mois.</p>
          </div>
          <p>Nous préparons votre espace personnel et vous enverrons vos accès dans les prochaines heures.</p>
          <p>L'équipe LodgePro</p>
        </div>
      </div>`;

    if (BREVO_KEY) {
      await sendEmail(ADMIN_EMAIL, `💳 Nouveau client LodgePro — ${formule} — ${customerEmail}`, htmlAdmin, BREVO_KEY);
      if (customerEmail) {
        await sendEmail(customerEmail, '🏖️ Votre abonnement LodgePro est confirmé !', htmlClient, BREVO_KEY);
      }
    }

    return { statusCode: 200, body: JSON.stringify({ received: true }) };

  } catch (err) {
    console.error('Webhook error:', err);
    return { statusCode: 400, body: `Webhook Error: ${err.message}` };
  }
};
